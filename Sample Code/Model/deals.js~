var moment = require("moment-timezone")
var utils = require("../../lib/utils");
var _ = require("underscore");
var async = require("async");
var config = require("../../server/config.json");
var ejs = require("ejs");
var LoopBackContext = require('loopback-context');
var pdf = require('html-pdf');
var geoip = require('geoip-lite');
'use strict';

module.exports = function(Deals) {
	// here are two types of deals type 1) Link Deal to My Shopping Cart(CART) 2) Pay at point of Sale(SALE)
	Deals.createDeal = function(body, res, cb){
		// console.log(body, "this is the request")
		if(!body.businessId){
			return res.status(400).send({status : 400,message : "Business id is required."}) 
		}
		Deals.app.models.Business.findOne({where : {id : body.businessId, status : 1}}, function(err, business){		
			if(err) return res.status(500).json({status : 500, message: err.message});
			if(business){
				Deals.app.models.Member.findOne({where : {id : business.userId}}, function(err, user){
					if(user.membershipId ==1){
						var sql = "SELECT * FROM deals WHERE businessid = "+body.businessId+" AND to_timestamp(createdat/1000)::date = now()::date AND deal.status != 0 ";

						Deals.dataSource.connector.query(sql, function(err, deals){
							if(err) return res.status(500).json({status : 500, message: err.message});
							if(deals && deals.length == 0){
								addDeal(body, user, res);
							}else{
								return res.status(403).json({status : 403, message : "You have taken basic subscription so you can add only one deal at a time"})
							}	
						})
					}else{
						addDeal(body, user, res);
					}
				})
			}else{
				return res.status(404).send({status : 404, message : "Business not found"});     
			}
		})
	}

	Deals.remoteMethod(
	'createDeal', 
    {
      description: 'Create deal',
      http: {path: '/create-deal', verb: 'post'},
      accepts: [{arg: 'body', type: 'object', 'http': {source: 'body'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    function addDeal(body, user, res)	{
    	var pattern = new RegExp('.*'+body.title+'.*', "i");
    	Deals.find({where : {title : pattern}}, function(err, deal){
    		if(err) return res.status(500).json({status : 500, message: err.message});
    		var c = "";
    		if(deal.length == 0){
    			c = utils.generateSlug(body.title);
    		}else if(deal.length == 1){
    			c = deal[0].slug + "-" + deal.length;

    		}else{
    			var d = deal[deal.length - 1].slug;
    			var b = d.split('-');
    			var e = b.slice(0, b.length - 1)
    			c = e.join("-") + "-" + (parseInt(b[b.length - 1]) + 1);
    		}
	    	var fileName = body.imageUrl.split('/');
			var a = fileName[fileName.length - 1];
			var deal = {
				businessId : body.businessId,
				title : body.title,
				description : body.description,
				seoTitle : body.seoTitle,
				seoDescription : body.seoDescription,
				tnc : body.tnc,
				imageName : a,
				isFeatured : body.isFeatured,
				categoryId : body.categoryId,
				stateId : body.stateId,
				cityId : body.cityId,
				startDate : body.startDate,
				expireDate : body.expireDate,
				dealOffered : body.dealOffered,
				retailValue : body.retailValue,
				dealPrice : body.dealPrice,
				offerPercentage : body.offerPercentage, 
				dealType : body.dealType,
				promoCode : body.promoCode || "",
				shoppingCartUrl : body.shoppingCartUrl || "",
				slug : c,
				status : 1,
				isSelected : false,
				createdAt : Date.now(),
				updatedAt : Date.now()	
			};
			if(user.membershipId == 1){
				deal.isSelected = true
			}
			Deals.create(deal, function(err, data){
				if(err) return res.status(500).json({status : 500, message: err.message});

				if(data){
					return res.status(200).send({status : 200, message : "Deal created successfully."})     
				}
			})
    	})
    }
	// here type are of three types NEWEST, FEATURED, OLDER 
    Deals.getDealsByType = function(userId, type, res, cb){
    	if(!type){
			return res.status(400).send({status : 400,message : "Type is required."}) 
		}

		

		var sql = 'SELECT deals.id, title, dealprice, retailvalue, offerpercentage, imagename, ' +
				  'dealoffered - redeemcount as "dealRemaining", deals.slug, business.address, isfeatured, location.name cityname ';
				 
		if(userId){
			sql += " ,(SELECT count(*) FROM savedeals WHERE userid = "+userId+" AND dealid = deals.id) as isSave "
		} 

		sql += 'FROM deals ' +
			   'INNER JOIN business ON business.id = deals.businessid ' +
			   'INNER JOIN location ON location.id = deals.cityid ' +
			   'WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.isselected = true AND (extract(epoch from startdate::date) * 1000) <= ' + Date.now() + ' ';
		if(type == "NEWEST"){
			
			sql += 'AND deals.createdat + 86400000 >= ' + Date.now();
		}else if(type == "FEATURED"){
			sql += 'AND isfeatured = true ';
		}else if(type == "OLDER"){
			
			sql += 'AND deals.createdat + 86400000 <= ' + Date.now();
		}

		
		sql += " LIMIT 25"; 
	
    	Deals.dataSource.connector.query(sql, function(err, deals){
    		if(err) return res.status(500).json({status : 500, message: err.message});

    		if(deals && deals.length != 0){
    			var sql_1 = "SELECT count(*) from deals WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.isselected = true AND (extract(epoch from startdate::date) * 1000) <= " + Date.now() + " ";
    			if(type == "NEWEST"){
					
					sql_1 += 'AND deals.createdat + 86400000 >= ' + Date.now();
				}else if(type == "FEATURED"){
					sql_1 += 'AND isfeatured = true ';
				}else if(type == "OLDER"){
			
					sql_1 += 'AND deals.createdat + 86400000 <= ' + Date.now();
				}
				Deals.dataSource.connector.query(sql_1, function(err, count){
					if(err) return res.status(500).json({status : 500, message : err.message})	
					var isMore = false;
					if(parseInt(count[0].count) > 25){
						isMore = true;
					}
	    			var newDeals = [];
					async.each(deals, function(deal, callback){
						var obj = {
							id : deal.id,
							title : deal.title,
							dealPrice : deal.dealprice,
							retailValue : deal.retailvalue,
							offerPercentage : deal.offerpercentage,
							imageName : config.s3bucket.cloudURL + deal.imagename,
							dealRemaining : deal.dealRemaining,
							// address : deal.address,
							address : deal.cityname,
							isSave : false,
							isFeatured : deal.isfeatured,
							slug : deal.slug
						}
						if(parseInt(deal.issave)){
							obj.isSave = true;
						}
						newDeals.push(obj)
						callback();						
					}, function(err, data){
						if(err) return res.status(500).json({status : 500, message : err.message});
						return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals, isMore : isMore}});
					})
				})
			}else{
				return res.status(404).send({status : 404, message : "No deals found"});     	
			}

    	})
    }

    Deals.remoteMethod(
	'getDealsByType', 
    {
      description: 'Get deal by type',
      http: {path: '/deal-by-type', verb: 'get'},
      accepts: [{arg: 'userId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'type', type: 'string', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    Deals.getDealsByCategory = function(categorySlug, currentPage, pageSize, res, cb){

		var skip = currentPage ? (currentPage - 1) * pageSize : 1;
		var limit = pageSize ? pageSize : 20;

    	var sql = 'SELECT deals.id, title, dealprice, retailvalue, offerpercentage, deals.imagename, ' +
				  'dealoffered - redeemcount as "dealRemaining", deals.slug, business.address, isfeatured, category.name categoryname ' +
				  'FROM deals ' +
				  'INNER JOIN business ON business.id = deals.businessid ' +
				  'INNER JOIN category ON category.id = deals.categoryid ' +
				  "WHERE deals.status = 1 AND category.slug = '" + categorySlug + "' OFFSET " + skip + " LIMIT " + limit;
			
    	Deals.dataSource.connector.query(sql, function(err, deals){
    		if(err) return res.status(500).json({status : 500, message: err.message});

    		if(deals && deals.length != 0){
    			var newDeals = [];
				async.each(deals, function(deal, callback){
					
						newDeals.push({
							id : deal.id,
							title : deal.title,
							dealPrice : deal.dealprice,
							retailValue : deal.retailvalue,
							offerPercentage : deal.offerpercentage,
							imageName : config.s3bucket.cloudURL + deal.imagename,
							dealRemaining : deal.dealRemaining,
							address : deal.address,
							isFeatured : deal.isfeatured,
							slug : deal.slug,
							categoryName : deal.categoryname
						})
						callback();						
					
				}, function(err, data){
					if(err) return res.status(500).json({status : 500, message : err.message});
					return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals}});
				})
				     
			}else{
				return res.status(404).send({status : 404, message : "No deals found"});     	
			}

    	})
    }

    Deals.remoteMethod(
	'getDealsByCategory', 
    {
      description: 'Get deal by category',
      http: {path: '/deal-by-category', verb: 'get'},
      accepts: [{arg: 'categorySlug', type: 'string', 'http': {source: 'query'}},
      			{arg: 'currentPage', type: 'number', 'http': {source: 'query'}},
      			{arg: 'pageSize', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });


    Deals.getDealDetailsHome = function(slug, userId, res, cb){
    	if(!slug){
			return res.status(400).send({status : 400,message : "Slug is required."}) 
		}

		var sql = 'SELECT deals.id, deals.title, deals.offerpercentage, deals.businessid, deals.description dealdescription, deals.viewscount, deals.imagename, deals.retailvalue, deals.expiredate, deals.dealprice, deals.dealoffered - deals.redeemcount dealremaining, deals.promocode, deals.shoppingcarturl, ' +
				  'deals.redeemcount, deals.dealtype, deals.tnc, b.name, b.description businessdescription, b.phoneno, b.websiteurl, b.address, b.landmark, b.videourl,b.facebooklink, b.twitterlink, b.googlelink, ' +
			 	  'b.instagramlink, b.yelplink, b.businessmainimage, b.slug businessslug, location.geo, b.email, b.avgrating, b.id businessid, b.menubrochure ';
		if(userId){
			sql += " ,(SELECT count(*) FROM savedeals WHERE userid = "+userId+" AND dealid = deals.id) as isSave "
		}

		sql += 'FROM deals '+
			   "INNER JOIN business b ON b.id = deals.businessid " +
			   "INNER JOIN location ON location.id = b.cityid " +
			   "WHERE deals.slug = '"+ slug +"'";
			
		Deals.dataSource.connector.query(sql, function(err, deals){
    		if(err) return res.status(500).json({status : 500, message: err.message});

    		if(deals && deals.length != 0){
    			var obj = {
    				id : deals[0].id,
    				title : deals[0].title,
    				dealDescription : deals[0].dealdescription,
    				imageName : config.s3bucket.cloudURL + deals[0].imagename,
    				retailValue : deals[0].retailvalue,
    				dealPrice : deals[0].dealprice,
    				dealType : deals[0].dealtype,
    				expireDate : deals[0].expiredate,
    				itemLeft : deals[0].dealremaining,
    				offerPercentage : deals[0].offerpercentage,
    				tnc : deals[0].tnc || "",
    				redeemCount : deals[0].redeemcount,
    				promoCode : deals[0].promocode || "",
    				shoppingCartUrl : deals[0].shoppingcarturl || "",
    				businessId : deals[0].businessid,
    				businessName : deals[0].name,
    				businessSlug : deals[0].businessslug,
    				businessMainImage : deals[0].businessmainimage,
    				businessDescription : deals[0].businessdescription,
    				businessLng : deals[0].geo.x,
    				businessLat : deals[0].geo.y,
    				businessEmail : deals[0].email,
    				businessMenuBrochure : config.s3bucket.cloudURL + deals[0].menubrochure,
    				phoneNo : deals[0].phoneno,
    				address : deals[0].address || "",
    				landmark : deals[0].landmark || "",
    				website : deals[0].websiteurl || "",
    				facebookLink : deals[0].facebooklink || "",
    				videoLink : deals[0].videourl || "",
    				twitterLink : deals[0].twitterlink || "",
    				googleLink : deals[0].googlelink || "",
    				instagramLink : deals[0].instagramlink || "",
    				yelpLink : deals[0].yelplink || "",
    				rating : Math.round(deals[0].avgrating),
    				isSave : false,
    				businessWorkingHours : [],
    				businessImages : []
    			};
    			if(parseInt(deals[0].issave)){
					obj.isSave = true;
				}
    			async.parallel([
				    function(callback) {
				        var sql_1 = "SELECT * FROM businessimage WHERE businessid = " + parseInt(deals[0].businessid) + " ORDER BY sequence";

		    			Deals.dataSource.connector.query(sql_1, function(err, businessImages){
		    				if(err) return res.status(500).json({status : 500, message: err.message});
		    				_.each(businessImages , function(businessImage){
		    					obj.businessImages.push({
		    						imageUrl : config.s3bucket.cloudURL + businessImage.imagename,
		    						sequence : businessImage.sequence	
		    					})
		    				})
		    				callback();
		    			})
				    },
				    function(callback) {
				        var sql_2 = "SELECT * FROM businessworkinghours WHERE businessid = " + parseInt(deals[0].businessid);
		    			Deals.dataSource.connector.query(sql_2, function(err, businessWorkingHours){
		    				if(err) return res.status(500).json({status : 500, message: err.message});	
		    				_.each(businessWorkingHours , function(businessWorkingHour){
		    					obj.businessWorkingHours.push({
		    						day : businessWorkingHour.day,
		    						from1 : businessWorkingHour.from1,
		    						to1 : businessWorkingHour.to1,
		    						from2 : businessWorkingHour.from2,
		    						to2 : businessWorkingHour.to2	
		    					 })
		    				})
		    				callback()
		    			})
				    }
				],
				function(err, results) {
					if(obj.businessWorkingHours.length != 0){
						var days = utils.compareDays(obj.businessWorkingHours);
						obj.businessWorkingHours = days;
					}
					Deals.updateAll({id : deals[0].id}, {viewsCount : deals[0].viewscount + 1}, function(err, count){
						if(err) return res.status(500).json({status : 500, message: err.message});	
						return res.status(200).send({status : 200, message : "Details of deal", data : {dealDetails : obj}});     
					})
				});
			}else{
				return res.status(404).send({status : 404, message : "No deal found"});     	
			}
    	})
    }

    Deals.remoteMethod(
	'getDealDetailsHome', 
    {
      description: 'Get deal by slug',
      http: {path: '/get-deal-slug', verb: 'get'},
      accepts: [{arg: 'slug', type: 'string', 'http': {source: 'query'}},
      			{arg: 'userId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    Deals.getRelatedDeals = function(slug, userId, res, cb){
    	if(!slug){
			return res.status(400).send({status : 400,message : "Slug is required."}) 
		}
		
		Deals.findOne({where : {slug : slug}}, function(err, deal){
			if(err) return res.status(500).json({status : 500, message: err.message});
			if(deal){
		    	var sql = 'SELECT deals.id, title, dealprice, retailvalue, offerpercentage, imagename, ' +
						  'dealoffered - redeemcount as "dealRemaining", deals.slug, business.address, isfeatured, location.name cityname '; 
				if(userId){
					sql += " ,(SELECT count(*) FROM savedeals WHERE userid = "+userId+" AND dealid = deals.id) as isSave "
				}	
				
				sql += 	'FROM deals ' +
						'INNER JOIN business ON business.id = deals.businessid ' +
						'INNER JOIN location ON location.id = business.cityid ' + 
 						"WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.slug != '"+slug+"' AND deals.businessid = " + deal.businessId + " AND (extract(epoch from startdate::date) * 1000) <= " +Date.now()+ " LIMIT 25";
		    	Deals.dataSource.connector.query(sql, function(err, deals){
		    		if(err) return res.status(500).json({status : 500, message: err.message});

		    		if(deals && deals.length != 0){
		    			var sql_1 = "SELECT count(*) from deals WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.slug != '"+slug+"' AND deals.businessid = " + deal.businessId +" AND (extract(epoch from startdate::date) * 1000) <= " + Date.now();
						Deals.dataSource.connector.query(sql_1, function(err, count){
							if(err) return res.status(500).json({status : 500, message : err.message})	
							var isMore = false;
							if(parseInt(count[0].count) > 25){
								isMore = true;
							}
			    			var newDeals = [];
							async.each(deals, function(deal, callback){
								var obj = {
									id : deal.id,
									title : deal.title,
									dealPrice : deal.dealprice,
									retailValue : deal.retailvalue,
									offerPercentage : deal.offerpercentage,
									imageName : config.s3bucket.cloudURL + deal.imagename,
									dealRemaining : deal.dealRemaining,
									address : deal.cityname,
									isFeatured : deal.isfeatured,
									slug : deal.slug,
									businessId : deal.businessId,
									isSave : false
								}
								if(parseInt(deal.issave)){
									obj.isSave = true;
								}
								newDeals.push(obj);
								callback();						
							}, function(err, data){
								if(err) return res.status(500).json({status : 500, message : err.message});
								return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals, isMore : isMore}});
							})
						})
					}else{
						return res.status(404).send({status : 404, message : "No related deals found"});     	
					}

		    	})
			}else{
				return res.status(404).send({status : 404, message : "Deal not found"});
			}
		})
    }

    Deals.remoteMethod(
	'getRelatedDeals', 
    {
      description: 'Get related deal by slug',
      http: {path: '/get-related-deals', verb: 'get'},
      accepts: [{arg: 'slug', type: 'string', 'http': {source: 'query'}},
      			{arg: 'userId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    }); 

    Deals.getDealsActivity = function(businessId, currentPage, pageSize, res, cb){
    	if(!businessId){
			return res.status(400).send({status : 400,message : "Business id is required."}) 
		}
		var skip = currentPage ? (currentPage - 1) * pageSize : 1;
		var limit = pageSize ? pageSize : 20;
		Deals.find({where : {businessId : businessId, status : {neq : 0}}, fields : ['id', 'title', 'viewsCount', 'redeemCount', 'shareCount', 'status', 'imageName'], skip : skip, limit : limit, order : 'createdAt DESC'}, function(err, deals){
			if(err) return res.status(500).json({status : 500, message : err.message});

			if(deals && deals.length != 0){
				Deals.count({businessId : businessId, status : {neq : 0}}, function(err, count){
					if(err) return res.status(500).json({status : 500, message : err.message});					
					deals = _.map(deals, function(deal){ 
						return {
							id : deal.id, 
							title : deal.title,
							imageUrl : config.s3bucket.cloudURL + deal.imageName,
							viewsCount : deal.viewsCount,
							redeemCount : deal.redeemCount,
							shareCount : deal.shareCount,
							status : deal.status
						}
					});
					return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : deals, totalCount : count}});
				})
			}else{
				return res.status(404).json({status : 404, message : "No deals found"});
			}	
		})
    } 

    Deals.remoteMethod(
	'getDealsActivity', 
    {
      description: 'Get deal activity of particular business',
      http: {path: '/get-deals-activity', verb: 'get'},
      accepts: [{arg: 'businessId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'currentPage', type: 'number', 'http': {source: 'query'}},
      			{arg: 'pageSize', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    Deals.getBusinessDeals = function(businessId, currentPage, pageSize, res, cb){
    	if(!businessId){
			return res.status(400).send({status : 400,message : "Business id is required."}) 
		}

		var skip = currentPage ? (currentPage - 1) * pageSize : 1;
		var limit = pageSize ? pageSize : 20;

		var sql = "SELECT deals.id, deals.title, deals.status, deals.slug, deals.startdate, deals.isfeatured, deals.businessid, deals.isselected, category.name " +
				  "FROM deals " +
				  "INNER JOIN category ON category.id = deals.categoryid " +
				  "WHERE businessid = "+businessId+" AND deals.status != 0 ORDER BY deals.createdat OFFSET " + skip + " LIMIT " + limit;

		var sql_1 = "SELECT count(*) " +
				    "FROM deals " +
				    "INNER JOIN category ON category.id = deals.categoryid " +
				    "WHERE businessid = "+businessId+" AND deals.status != 0 ";

		Deals.dataSource.connector.query(sql_1, function(err, totalCount){
			if(err) return res.status(500).json({status : 500, message : err.message});
			Deals.dataSource.connector.query(sql, function(err, deals){
				if(err) return res.status(500).json({status : 500, message : err.message});

				if(deals && deals.length != 0){
					var newDeals = [];
					async.each(deals, function(deal, callback){
						newDeals.push({
							id : deal.id,
							title : deal.title,
							status : deal.status,
							slug : deal.slug,
							categoryName : deal.name,
							businessId : deal.businessid,
							isSelected : deal.isselected,
							isFeatured : deal.isfeatured,
							startDate :  moment(deal.startdate).format("MMM DD")
						})
						callback();						
					}, function(err, data){
						if(err) return res.status(500).json({status : 500, message : err.message});
						return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals, totalCount : totalCount[0].count}});
					})
				}else{
					return res.status(404).json({status : 404, message : "No deals found"});
				}	
			})
		})

    }

    Deals.remoteMethod(
	'getBusinessDeals', 
    {
      description: 'Get deals of particular business',
      http: {path: '/get-business-deals', verb: 'get'},
      accepts: [{arg: 'businessId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'currentPage', type: 'number', 'http': {source: 'query'}},
      			{arg: 'pageSize', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });


    Deals.getDealDetailsBusiness = function(dealId, businessId, res, cb){
    	if(!dealId){
			return res.status(400).send({status : 400,message : "Deal id is required."}) 
		}

		Deals.app.models.Business.findOne({where : {id : businessId}}, function(err, business){
			if(err) return res.status(500).json({status : 500, message : err.message})

			if(business){
				Deals.findOne({where : {and :[{id : dealId}, {businessId : businessId}]}}, function(err, deal){
					if(err) return res.status(500).json({status : 500, message : err.message})
					if(deal){
						var obj = {
							id : deal.id,
							businessName : business.name,
							title : deal.title,
							description : deal.description,
							seoTitle : deal.seoTitle,
							seoDescription : deal.seoDescription,
							tnc : deal.tnc,
							imageUrl : config.s3bucket.cloudURL + deal.imageName,
							isFeatured : deal.isFeatured,
							categoryId : deal.categoryId,
							stateId : deal.stateId,
							cityId : deal.cityId,
							startDate : deal.startDate,
							expireDate : deal.expireDate,
							dealOffered : deal.dealOffered,
							dealUsed : deal.dealUsed,
							dealType : deal.dealType,
							retailValue : deal.retailValue,
							dealPrice : deal.dealPrice,
							offerPercentage : deal.offerPercentage,
							promoCode : deal.promoCode,
							shoppingCartUrl : deal.shoppingCartUrl
						};
						return res.status(200).json({status : 200, message : "Deal details", data : {deal : obj}})
					}else{
						return res.status(404).json({status : 404, message : "Deal not found"})
					}
				})
			}else{
				return res.status(404).json({status : 404, message : "Business not found"})
			}
		})	
    }


    Deals.remoteMethod(
    'getDealDetailsBusiness',
    {
    	description: 'Get deals details for edit the deal',
      	http: {path: '/vendor/deal', verb: 'get'},
      	accepts: [{arg: 'dealId', type: 'number', 'http': {source: 'query'}},
      			  {arg: 'businessId', type: 'number', 'http': {source: 'query'}},	
      			  {arg: 'res', type: 'object', 'http': {source: 'res'}}],
      	returns: {arg: 'response', type: 'string'}
    })

    Deals.updateStatus = function(body, res, cb){
    	// console.log(dealId, businessId, isActive);
		if(!body.dealId){
			return res.status(400).send({status : 400,message : "Deal id is required."}) 
		}
		if(!body.businessId){
			return res.status(400).send({status : 400,message : "Business id is required."}) 
		}
		Deals.findOne({where : {and : [{id : body.dealId}, {businessId : body.businessId}]}}, function(err, deal){
			if(err) return res.status(500).json({status : 500, message : err.message})
			if(deal.status == 1 && body.isActive == true){

			}
			if(deal){
				if(body.isActive){
					// This is called when you want to active the deactivated category
					Deals.updateAll({id : body.dealId}, {status : 1, updatedAt : Date.now()}, function(err, data){
						if(err) return res.status(500).send({status : 500,message : err.message})  
			    		return res.status(200).send({status : 200, message : "Deals activated successfully"});	
					})
				}else{
					// This is called when you want to deactivated the active category
					Deals.updateAll({id : body.dealId}, {status : 2, isSelected : false, updatedAt : Date.now()}, function(err, data){
						if(err) return res.status(500).send({status : 500,message : err.message})  
			    		return res.status(200).send({status : 200, message : "Deals deactivated successfully"});	
					})			
				}    	
			}else{
				return res.status(404).json({status : 404, message : "Deal not found"})
			}
		})
    }


    Deals.remoteMethod(
	'updateStatus', 
    {
      description: 'Activate and deactivate deal',
      http: {path: '/deal/status', verb: 'put'},
      accepts: [{arg: 'body', type: 'object', 'http': {source: 'body'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });


    Deals.updateDeal = function(body, res, cb){
		if(!body.dealId){
			return res.status(400).send({status : 400,message : "Deal id is required."}) 
		}

		Deals.findOne({where : {id : body.dealId}}, function(err, deal){
			if(err) return res.status(500).json({status : 500, message : err.message});
			if(deal){
				var fileName = body.imageUrl.split('/');
				var a = fileName[fileName.length - 1];
				var deal_1 = {
					title : body.title,
					description : body.description,
					seoTitle : body.seoTitle,
					seoDescription : body.seoDescription,
					tnc : body.tnc,
					imageName : a,
					isFeatured : body.isFeatured,
					categoryId : body.categoryId,
					stateId : body.stateId,
					cityId : body.cityId,
					startDate : body.startDate,
					expireDate : body.expireDate,
					dealOffered : body.dealOffered,
					retailValue : body.retailValue,
					dealPrice : body.dealPrice,
					offerPercentage : body.offerPercentage,
					dealType : body.dealType,
					promoCode : body.promoCode || "",
					shoppingCartUrl : body.shoppingCartUrl || "",
					updatedAt : Date.now()	
				};

				if(deal.title != body.title){
					var pattern = new RegExp('.*'+body.title+'.*', "i");
					Deals.find({where : {title : pattern}, order : 'slug'}, function(err, deals){
    					if(err) return res.status(500).json({status : 500, message: err.message});
						var c = "";
			    		if(deals.length == 0){
			    			c = utils.generateSlug(body.title);
			    		}else if(deals.length == 1){
			    			c = deals[0].slug + "-" + deals.length;

			    		}else{
			    			var d = deals[deals.length - 1].slug;
			    			var b = d.split('-');
			    			var e = b.slice(0, b.length - 1)
			    			c = e.join("-") + "-" + (parseInt(b[b.length - 1]) + 1);
			    		}
	    				deal_1.slug = c;
	    				Deals.updateAll({id : body.dealId}, deal_1, function(err){
							if(err) return res.status(500).send({status : 500,message : err.message})
							return res.status(200).send({status : 200, message : "Deal updated successfully"});	  
						})
    				})
				}else{
					Deals.updateAll({id : body.dealId}, deal_1, function(err){
						if(err) return res.status(500).send({status : 500,message : err.message})
						return res.status(200).send({status : 200, message : "Deal updated successfully"});	  
					})					
				}
			}else{
				return res.status(404).json({status : 404, message : "Deal not found"})
			}
		})

    }

    Deals.remoteMethod(
	'updateDeal', 
    {
      description: 'update deal',
      http: {path: '/deal', verb: 'put'},
      accepts: [{arg: 'body', type: 'object', 'http': {source: 'body'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });


    Deals.deleteDeal = function(dealId, businessId, res, cb){
		if(!dealId){
			return res.status(400).send({status : 400,message : "Deal id is required."}) 
		}
		Deals.findOne({where : {and : [{id : dealId}, {businessId : businessId}]}}, function(err, deal){
			if(err) return res.status(500).json({status : 500, message : err.message})

			if(deal){
				// This is called when you want to deactivated the active category
				Deals.updateAll({id : dealId}, {status : 0, isSelected : false, updatedAt : Date.now()}, function(err, data){
					if(err) return res.status(500).send({status : 500,message : err.message})  
		    		return res.status(200).send({status : 200, message : "Deal deleted successfully"});	
				})				
			}else{
				return res.status(404).json({status : 404, message : "Deal not found"})
			}
		})
    }


    Deals.remoteMethod(
	'deleteDeal', 
    {
      description: 'Delete deal',
      http: {path: '/deal', verb: 'delete'},
      accepts: [{arg: 'dealId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'businessId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

   
    Deals.redeemDeal = function(req, res, cb){
    	
    	if(!req.body.dealId){
    		return res.status(400).json({status : 400, message : "Deal id is required"})
    	}

    	if(req.body.isShare != true && req.body.isShare != false){
    		return res.status(400).json({status : 400, message : "Is share is required"})	
    	}

    	var ip = utils.getCallerIP(req);
    	var filter = {where : {and : [{dealId : req.body.dealId}, {userIp : ip}]}}

    	if(req.body.userId){
    		filter.where.and.push({userId : req.body.userId});
    	}

    	Deals.app.models.Redeem.find(filter, function(err, redeems){
    		if(err) return res.status(500).json({status : 500, message : err.message})
    		redeemByType(req, res, redeems)
    	})
    }

    Deals.remoteMethod(
    'redeemDeal',{
    	description: 'Redeem deal either by share or by normal',
      http: {path: '/redeem', verb: 'post'},
      accepts: [{arg: 'req', type: 'object', 'http': {source: 'req'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    })

    function redeemByType(req, res, count){
    	var ip = utils.getCallerIP(req);
    	if(req.body.type == "CART"){
    		if(count && count.length > 0){
    			return res.status(200).json({status : 200, message : "Deal redeemed successfully"})
    		}else{
	    		Deals.findOne({where : {id : req.body.dealId}}, function(err, deal){
					if(err) return res.status(500).json({status : 500, message : err.message})
						var obj = {
							redeemCount : deal.redeemCount + 1
						}
						if(req.body.isShare == true){
							obj.shareCount = deal.shareCount + 1;
						}
					Deals.updateAll({id : req.body.dealId}, obj, function(err){
						if(err) return res.status(500).json({status : 500, message : err.message})
						var redeem = {
							userId : req.body.userId,
							dealId : req.body.dealId,
							userIp : ip,
							createdAt : Date.now(),
							updatedAt : Date.now()
						}
						Deals.app.models.Redeem.create(redeem, function(err, data){
							if(err) return res.status(500).json({status : 500, message : err.message})
							return res.status(200).json({status : 200, message : "Deal redeemed successfully"})
						})
						
					})
				})
    		}
    	}else{
    		if(count && count.length > 0){
    			generatePdf(req.body.dealId, req.body.userId, ip, req.body.isShare, res);
    		}else{
    			Deals.findOne({where : {id : req.body.dealId}}, function(err, deal){
    				if(err) return res.status(500).json({status : 500, message : err.message})
    					var obj = {
    						redeemCount : deal.redeemCount + 1
    					}
    					if(req.body.isShare == true){
    						obj.shareCount = deal.shareCount + 1;
    					}
    				Deals.updateAll({id : req.body.dealId}, obj, function(err){
    					if(err) return res.status(500).json({status : 500, message : err.message})
    					generatePdf(req.body.dealId, req.body.userId, ip, req.body.isShare, res)
    				})
    			})
    		}
    	}
    }

    function generatePdf(dealId, userId, ip, isShare, res){
    	var sql = ""; 
   
		if(userId){
			sql += "SELECT deals.title dealtitle, business.name businessname, business.address businessaddress, deals.retailvalue, deals.offerpercentage, " +
				   "dealoffered - redeemcount as itemleft, deals.description, deals.dealprice, deals.isfeatured, " +
				   "deals.expiredate, deals.startdate, concat_ws(' ', firstname, lastname) fullname from deals " +
				   "INNER JOIN business ON business.id = deals.businessid " +
				   "INNER JOIN member ON member.id = " + userId + " " +
				   "WHERE deals.id = " + dealId;
		}else{
			sql+= "SELECT deals.title dealtitle, business.name businessname, business.address businessaddress, deals.retailvalue, deals.offerpercentage, " +
				  "dealoffered - redeemcount as itemleft, deals.description, deals.dealprice, deals.isfeatured, " +
				  "deals.expiredate, deals.startdate from deals " +
				  "INNER JOIN business ON business.id = deals.businessid " +
				  "WHERE deals.id = " + dealId;
		}		  

		Deals.dataSource.connector.query(sql, function(err, data){
			if(err) return res.status(500).json({status : 500, message : err.message})

			var obj = {
				dealTitle : data[0].dealtitle,
				businessName : data[0].businessname,
				businessAddress : data[0].businessaddess,
				regularPrice : data[0].retailvalue,
				itemLeft : data[0].itemleft,
				dealDescription : data[0].description,
				validityStartedOn : data[0].startdate,
				expireDate : data[0].expiredate,
				fullName : data[0].fullname
			};
			if(isShare == true && data[0].isfeatured == true){
				obj.discountedPrice = ((parseInt(data[0].offerpercentage) + 10) * parseInt(data[0].retailvalue))/100;
			}else{
				obj.discountedPrice = data[0].dealprice;
			}
			var redeem = {
				userId : userId,
				dealId : dealId,
				userIp : ip,
				createdAt : Date.now(),
				updatedAt : Date.now()
			}

			ejs.renderFile('./server/views/pdf.ejs', {data : obj}, function(err, html) {
			    
			    if (html) {
			        res.contentType("application/pdf");				    
			        pdf.create(html).toStream((err, pdfStream) => {
					    if (err) {   
					     
					      return res.sendStatus(500)
					    } else {
					      res.statusCode = 200             

					      
					        Deals.app.models.Redeem.create(redeem, function(err, data){
								if(err) return res.status(500).json({status : 500, message : err.message})
					      		pdfStream.pipe(res);
								
							})

					    }
					})			       	
			    }
			   
			    else {
			       return res.end('An error occurred');
			       console.log(err);
			    }
			});

		})
    }


    Deals.searchDeals = function(req, res, cb){

    	var skip = req.query.currentPage ? (req.query.currentPage - 1) * req.query.pageSize : 1;
		var limit = req.query.pageSize ? req.query.pageSize : 20;
    	if(req.query.locationSlug){
    		Deals.app.models.Location.findOne({where : {slug : req.query.locationSlug}}, function(err, location){
	    		if(err) return res.status(500).json({status : 500, message : err.message})
	    		if(location){
	    			var sql = "SELECT  deals.id, title, dealprice, retailvalue, offerpercentage, deals.imagename, " +
							  "dealoffered - redeemcount as dealremaining, deals.slug, business.address, isfeatured, category.name ";
				    if(req.query.userId){
						sql += " ,(SELECT count(*) FROM savedeals WHERE userid = "+req.query.userId+" AND dealid = deals.id) as isSave "
					}
					sql += "FROM deals " +
						   "INNER JOIN business ON deals.businessid = business.id " +
						   "INNER JOIN category ON deals.categoryid = category.id " +
						   "WHERE deals.cityId = "+location.id+" AND deals.status = 1 AND deals.dealoffered > 0 AND deals.isselected = true ";		  
					if(req.query.categorySlug != "all" && req.query.categorySlug != undefined){
						sql += "AND category.slug = '" + req.query.categorySlug + "' ";
					}		  

					if(req.query.type == "newest"){
						sql += 'AND deals.createdat + 86400000 >= ' + Date.now() + ' ';
						if(req.query.filter == "FEATURED"){
							sql += 'AND isfeatured = true ';
						}
					}else if(req.query.type == "featured"){
						sql += 'AND isfeatured = true ';
					}else if(req.query.type == "older"){
						sql += 'AND deals.createdat + 86400000 <= ' + Date.now() + ' ';
						if(req.query.filter == "FEATURED"){
							sql += 'AND isfeatured = true ';
						}
					}else if(req.query.type == ''){
						if(req.query.filter == "FEATURED"){
							sql += 'AND isfeatured = true ';
						}
					}

					sql += "AND (extract(epoch from startdate::date) * 1000) <= " + Date.now() + " ";

					if(req.query.filter == "LOW2HIGH"){
						sql += "ORDER BY dealprice ASC ";
					}else if(req.query.filter == "HIGH2LOW"){
						sql += "ORDER BY dealprice DESC ";
					}else if(req.query.filter == "A2Z"){
						sql += "ORDER BY deals.title ASC ";
					}else if(req.query.filter == "Z2A"){
						sql += "ORDER BY deals.title DESC ";
					}else{
						sql += "ORDER BY deals.createdat DESC ";
					}

					sql += "OFFSET " + skip + " LIMIT " + limit;
					var sql_1 = "SELECT count(*) " +
								"FROM deals " +
						        "INNER JOIN business ON deals.businessid = business.id " +
						        "INNER JOIN category ON deals.categoryid = category.id " +
						        "WHERE deals.cityId = "+location.id+" AND deals.status = 1 AND deals.dealoffered > 0 AND deals.isselected = true ";		  
					if(req.query.categorySlug != "all" && req.query.categorySlug != undefined){
						sql_1 += "AND category.slug = '" + req.query.categorySlug + "' ";
					}		  

					if(req.query.type == "newest"){
						sql_1 += 'AND deals.createdat + 86400000 >= ' + Date.now() + ' ';
						if(req.query.filter == "FEATURED"){
							sql_1 += 'AND isfeatured = true ';
						}
					}else if(req.query.type == "featured"){
						sql_1 += 'AND isfeatured = true ';
					}else if(req.query.type == "older"){
						sql_1 += 'AND deals.createdat + 86400000 <= ' + Date.now() + ' ';
						if(req.query.filter == "FEATURED"){
							sql_1 += 'AND isfeatured = true ';
						}
					}

					sql_1 += "AND (extract(epoch from startdate::date) * 1000) <= " + Date.now();
					
					Deals.dataSource.connector.query(sql_1, function(err, totalCount){
						
						if(err) return res.status(500).json({status : 500, message : err.message})
						Deals.dataSource.connector.query(sql, function(err, deals){
							if(err) return res.status(500).json({status : 500, message : err.message})
							if(deals.length != 0){
								var newDeals = [];
								async.each(deals, function(deal, callback){
									var obj = {
										id : deal.id,
										title : deal.title,
										dealPrice : deal.dealprice,
										retailValue : deal.retailvalue,
										offerPercentage : deal.offerpercentage,
										imageName : config.s3bucket.cloudURL + deal.imagename,
										dealRemaining : deal.dealremaining,
										// address : deal.address,
										address : location.name,
										isFeatured : deal.isfeatured,
										slug : deal.slug,
										categoryName : deal.name || "",
										businessId : deal.businessId,
										isSave : false
									}
									if(parseInt(deal.issave)){
										obj.isSave = true;
									}
									newDeals.push(obj);
									callback();						
								}, function(err, data){
									if(err) return res.status(500).json({status : 500, message : err.message});
									return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals, totalCount : parseInt(totalCount[0].count)}});
								})
							}else{
								return res.status(404).json({status : 404, message : "No deals found"})		
							}
						})
					})
	    		}else{
	    			return res.status(404).json({status : 404, message : "No deals found"})
	    		}
	    	})
    	}else{
    
    		 var ip = utils.getCallerIP(req)
  			var geo = geoip.lookup(ip);

  			var sql_1 = "SELECT " + 
						"ST_Distance( " +
						"(ST_GeomFromText(concat('POINT(',split_part(trim(location.geo::text, '()'), ',', 1)::float,' ', split_part(trim(location.geo::text, '()'), ',', 2)::float,')'), 4326)), " +
						"ST_SetSRID(ST_Point("+geo.ll[1]+","+geo.ll[0]+"),4326)::geography(point,4326) " +
						")/1000 as distance, " +
						"deals.id, title, dealprice, retailvalue, offerpercentage, deals.imagename, " +
					    "dealoffered - redeemcount as dealremaining, deals.slug, business.address, isfeatured, category.name, location.name cityname ";
			if(req.query.userId){
				sql_1 += " ,(SELECT count(*) FROM savedeals WHERE userid = "+req.query.userId+" AND dealid = deals.id) as isSave "
			}
			sql_1 += "FROM deals " +
				     "INNER JOIN business ON deals.businessid = business.id " +
				     "INNER JOIN category ON deals.categoryid = category.id " +
				     "INNER JOIN location ON location.id = deals.cityid " +
				     "WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.isselected = true ";

			if(req.query.categorySlug != "all" && req.query.categorySlug != undefined){
				sql_1 += "AND category.slug = '"+req.query.categorySlug+"' ";
			}

			if(req.query.type == "newest"){
				sql_1 += 'AND deals.createdat + 86400000 >= ' + Date.now() + ' ';
				if(req.query.filter == "FEATURED"){
					sql_1 += 'AND isfeatured = true ';
				}
			}else if(req.query.type == "featured"){
				sql_1 += 'AND isfeatured = true ';
			}else if(req.query.type == "older"){
				sql_1 += 'AND deals.createdat + 86400000 <= ' + Date.now() + ' ';
				if(req.query.filter == "FEATURED"){
					sql_1 += 'AND isfeatured = true ';
				}
			}else if(req.query.type == ''){
				if(req.query.filter == "FEATURED"){
					sql_1 += 'AND isfeatured = true ';
				}
			}

			sql_1 += "AND (extract(epoch from startdate::date) * 1000) <= " + Date.now() + " ORDER BY distance "; 
		
			if(req.query.filter == "LOW2HIGH"){
				sql_1 += ", dealprice ASC ";
			}else if(req.query.filter == "HIGH2LOW"){
				sql_1 += ", dealprice DESC ";
			}else if(req.query.filter == "A2Z"){
				sql_1 += ", deals.title ASC ";
			}else if(req.query.filter == "Z2A"){
				sql_1 += ", deals.title DESC ";
			}
			sql_1 += "OFFSET " + skip + " LIMIT " + limit;
			
			var sql_2 = "SELECT count(*)" + 
						"FROM deals " +
					    "INNER JOIN business ON deals.businessid = business.id " +
					    "INNER JOIN category ON deals.categoryid = category.id " +
					    "INNER JOIN location ON location.id = deals.cityid " +
					    "WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.isselected = true ";

			if(req.query.categorySlug != "all" && req.query.categorySlug != undefined){
				sql_2 += "AND category.slug = '"+req.query.categorySlug+"' ";
			}

			if(req.query.type == "newest"){
				sql_2 += 'AND deals.createdat + 86400000 >= ' + Date.now() + ' ';
				if(req.query.filter == "FEATURED"){
					sql_2 += 'AND isfeatured = true ';
				}
			}else if(req.query.type == "featured"){
				sql_2 += 'AND isfeatured = true ';
			}else if(req.query.type == "older"){
				sql_2 += 'AND deals.createdat + 86400000 <= ' + Date.now() + ' ';
				if(req.query.filter == "FEATURED"){
					sql_2 += 'AND isfeatured = true ';
				}
			}

			sql_2 += "AND (extract(epoch from startdate::date) * 1000) <= " + Date.now();
			Deals.dataSource.connector.query(sql_2, function(err, totalCount){
	    		if(err) return res.status(500).json({status : 500, message: err.message});
				Deals.dataSource.connector.query(sql_1, function(err, deals_1){
		    		if(err) return res.status(500).json({status : 500, message: err.message});

		    		if(deals_1 && deals_1.length != 0){
		    			var newDeals_1 = [];
						async.each(deals_1, function(deal_1, callback){
							var obj = {
								id : deal_1.id,
								title : deal_1.title,
								dealPrice : deal_1.dealprice,
								retailValue : deal_1.retailvalue,
								offerPercentage : deal_1.offerpercentage,
								imageName : config.s3bucket.cloudURL + deal_1.imagename,
								dealRemaining : deal_1.dealremaining,
								address : deal_1.cityname,
								isFeatured : deal_1.isfeatured,
								slug : deal_1.slug,
								categoryName : deal_1.name || "",
								businessId : deal_1.businessId,
								isSave : false
							}
							if(parseInt(deal_1.issave)){
								obj.isSave = true;
							}
							newDeals_1.push(obj)
							callback();						
						}, function(err, data){
							if(err) return res.status(500).json({status : 500, message : err.message});
							return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals_1, totalCount : parseInt(totalCount[0].count)}});
						})
					}else{
						return res.status(404).send({status : 404, message : "No deals found"});     	
					}
		    	})
			})
    	}
    }

    Deals.remoteMethod(
    'searchDeals',
    {
    	description: 'Get deal according to search',
      	http: {path: '/search', verb: 'get'},
      	accepts: [{arg: 'req', type: 'object', 'http': {source: 'req'}},
  			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      	returns: {arg: 'response', type: 'string'}
    })


    Deals.selectDeals = function(body, res, cb){
    	if(!body.businessId){
    		return res.status(400).json({status : 400, message : "Business id is required"})
    	}

    	if(!body.dealId){
    		return res.status(400).json({status : 400, message : "Deal id is required"})
    	}
    	if(body.isSelect){
    		
    		Deals.find({where : {businessId : body.businessId, isSelected : true, status : 1}}, function(err, deals){
    			if(err) return res.status(500).json({status : 500, message : err.message});
    			if(deals.length < 4){
		    		Deals.updateAll({id : body.dealId}, {isSelected : true}, function(err){
		    			if(err) return res.status(500).json({status : 500, message : err.message});
		    			return res.status(200).json({status : 200, message : "Deals selected successfully"})
		    		})
    			}else{
    				return res.status(409).json({status : 409, message : "You have already selected four deals"})
    			}
    		})
    	}else{
    		Deals.updateAll({id : body.dealId}, {isSelected : false}, function(err){
    			if(err) return res.status(500).json({status : 500, message : err.message});
    			return res.status(200).json({status : 200, message : "Deals deselected successfully"})
    		}) 	
    	}
    }

    Deals.remoteMethod(
    'selectDeals',
    {
    	description: 'Select the deals which are to be shown on home screen',
      	http: {path: '/select', verb: 'put'},
      	accepts: [{arg: 'body', type: 'object', 'http': {source: 'body'}},
  			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      	returns: {arg: 'response', type: 'string'}
    })


    Deals.getBusinessRelatedDeals = function(slug, userId, res, cb){
    	if(!slug){
			return res.status(400).send({status : 400,message : "Slug is required."}) 
		}
	
		Deals.app.models.Business.findOne({where : {slug : slug}}, function(err, business){
			if(err) return res.status(500).json({status : 500, message: err.message});
			if(business){
		    	var sql = 'SELECT deals.id, title, dealprice, retailvalue, offerpercentage, imagename, ' +
						  'dealoffered - redeemcount as "dealRemaining", deals.slug, isfeatured, location.name cityname '; 
				if(userId){
					sql += " ,(SELECT count(*) FROM savedeals WHERE userid = "+userId+" AND dealid = deals.id) as isSave "
				}	
				
				sql += 	'FROM deals ' +
						'INNER JOIN location ON location.id = ' + business.cityId + " " +  
 						"WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.businessid = " + business.id + " AND (extract(epoch from startdate::date) * 1000) <= " +Date.now()+ " LIMIT 25";
		    	Deals.dataSource.connector.query(sql, function(err, deals){
		    		if(err) return res.status(500).json({status : 500, message: err.message});

		    		if(deals && deals.length != 0){
		    			var sql_1 = "SELECT count(*) from deals WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.businessid = " + business.id +" AND (extract(epoch from startdate::date) * 1000) <= " + Date.now();
						Deals.dataSource.connector.query(sql_1, function(err, count){
							if(err) return res.status(500).json({status : 500, message : err.message})	
							var isMore = false;
							if(parseInt(count[0].count) > 25){
								isMore = true;
							}
			    			var newDeals = [];
							async.each(deals, function(deal, callback){
								var obj = {
									id : deal.id,
									title : deal.title,
									dealPrice : deal.dealprice,
									retailValue : deal.retailvalue,
									offerPercentage : deal.offerpercentage,
									imageName : config.s3bucket.cloudURL + deal.imagename,
									dealRemaining : deal.dealRemaining,
									address : deal.cityname,
									isFeatured : deal.isfeatured,
									slug : deal.slug,
									businessId : deal.businessId,
									isSave : false
								}
								if(parseInt(deal.issave)){
									obj.isSave = true;
								}
								newDeals.push(obj);
								callback();						
							}, function(err, data){
								if(err) return res.status(500).json({status : 500, message : err.message});
								return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals, isMore : isMore}});
							})
						})
					}else{
						return res.status(404).send({status : 404, message : "No related deals found"});     	
					}

		    	})
			}else{
				return res.status(404).send({status : 404, message : "Deal not found"});
			}
		})
    }

    Deals.remoteMethod(
	'getBusinessRelatedDeals', 
    {
      description: 'Get business deals after business details on home screen',
      http: {path: '/business', verb: 'get'},
      accepts: [{arg: 'slug', type: 'string', 'http': {source: 'query'}},
      			{arg: 'userId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    Deals.allRelatedDeals = function(businessSlug, userId, currentPage, pageSize, res, cb){ 
    	if(!businessSlug){
    		return res.status(400).json({status : 400, message : "Slug is required"})
    	}
    	var skip = currentPage ? (currentPage - 1) * pageSize : 1;
		var limit = pageSize ? pageSize : 20;
    	Deals.app.models.Business.findOne({where : {slug : businessSlug}}, function(err, business){
    		if(err) return res.status(500).json({status : 500, message : err.message})

    		if(business){
    			var sql = 'SELECT deals.id, title, dealprice, retailvalue, offerpercentage, imagename, ' +
						  'dealoffered - redeemcount as "dealRemaining", deals.slug, isfeatured, location.name cityname '; 
				if(userId){
					sql += " ,(SELECT count(*) FROM savedeals WHERE userid = "+userId+" AND dealid = deals.id) as isSave "
				}	
				
				sql += 	'FROM deals ' +
						'INNER JOIN location ON location.id = ' + business.cityId + " " +  
 						"WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.businessid = " + business.id + " AND (extract(epoch from startdate::date) * 1000) <= " +Date.now()+ " OFFSET "+skip+" LIMIT " + limit;
		    	Deals.dataSource.connector.query(sql, function(err, deals){
		    		if(err) return res.status(500).json({status : 500, message: err.message});

		    		if(deals && deals.length != 0){
		    			var sql_1 = "SELECT count(*) from deals WHERE deals.status = 1 AND deals.dealoffered > 0 AND deals.businessid = " + business.id +" AND (extract(epoch from startdate::date) * 1000) <= " + Date.now();
						Deals.dataSource.connector.query(sql_1, function(err, count){
							if(err) return res.status(500).json({status : 500, message : err.message})	
			    			var newDeals = [];
							async.each(deals, function(deal, callback){
								var obj = {
									id : deal.id,
									title : deal.title,
									dealPrice : deal.dealprice,
									retailValue : deal.retailvalue,
									offerPercentage : deal.offerpercentage,
									imageName : config.s3bucket.cloudURL + deal.imagename,
									dealRemaining : deal.dealRemaining,
									address : deal.cityname,
									isFeatured : deal.isfeatured,
									slug : deal.slug,
									businessId : deal.businessId,
									isSave : false
								}
								if(parseInt(deal.issave)){
									obj.isSave = true;
								}
								newDeals.push(obj);
								callback();						
							}, function(err, data){
								if(err) return res.status(500).json({status : 500, message : err.message});
								return res.status(200).json({status : 200, message : "List of deals", data : {dealsList : newDeals, businessName : business.name, totalCount : parseInt(count[0].count)}});
							})
						})
					}else{
						return res.status(404).send({status : 404, message : "No related deals found"});     	
					}

		    	})																																																																																																																																																																																																																																																																																																					
    		}else{
    			return res.status(404).json({status : 404, message : "Business not found"})
    		}
    	})
    }

    Deals.remoteMethod(
	'allRelatedDeals', 
    {
      description: 'Get all business related deals ',
      http: {path: '/all', verb: 'get'},
      accepts: [{arg: 'slug', type: 'string', 'http': {source: 'query'}},
      			{arg: 'userId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'currentPage', type: 'number', 'http': {source: 'query'}},
      			{arg: 'pageSize', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    Deals.getCustomerDeals = function(customerId, currentPage, pageSize, res, cb){
    	if(!customerId){
    		return res.status(400).json({status : 400, message : "Customer id is required"})
    	}
    	var skip = currentPage ? (currentPage - 1) * pageSize : 1;
		var limit = pageSize ? pageSize : 20;
    	Deals.app.models.SaveDeals.find({where : {userId : customerId}}, function(err, deals){
    		if(err) return res.status(500).json({status : 500, message : err.message})
    		if(deals.length != 0){
    		var customerDeals = [];
    		Deals.app.models.SaveDeals.count({userId : customerId}, function(err, count){
    			if(err) return res.status(500).json({status : 500, message : err.message})
	    		async.each(deals, function(deal, callback){
	    			var sql = "SELECT business.name, deals.id, deals.dealtype, deals.isfeatured, deals.title, deals.expiredate, deals.status, dealoffered - redeemcount itemleft, " +
	    					  "(SELECT COUNT(*) FROM redeem WHERE userid = " + customerId + " AND dealid = deals.id), " + 
	    					  "((extract(epoch from expiredate::date) * 1000) + 86400000) expiredate " +
	    			          "FROM deals " + 
							  "INNER JOIN business ON business.id = deals.businessid " + 
							  "WHERE deals.id = " + deal.dealId + " AND deals.status != 0 OFFSET " + skip + " LIMIT " + limit;
						
					Deals.dataSource.connector.query(sql, function(err, data){
						if(err) return res.status(500).json({status : 500, message : err.message})
							
						if(data.length != 0){
							var obj = {
								dealId : data[0].id,
								businessName : data[0].name,
								dealTitle : data[0].title,
								isFeatured : data[0].isfeatured,
								expireDate : moment(data[0].expiredate).format("DD-MM-YYYY"),
								isAvailable : true,
								isUse : false,
								daysRemaining : 0,
								saveDealId : deal.id,
								dealType : data[0].dealtype
							}
							if(data[0].count > 0){
								obj.isUse = true;
							}
							if(data[0].itemleft <= 0 || data[0].expiredate <= Date.now()){
								obj.isAvailable = false
							}
							if(data[0].expiredate <= Date.now()){
								var a = Date.now() - parseInt(data[0].expiredate)
								if(a>0){
									obj.daysRemaining = a/86400000;
								}
							}
							customerDeals.push(obj);
							callback()
						}else{
							callback()
						}
					})	  
	    		}, function(err){
	    			return res.status(200).json({status : 200, message : "Deal list", data : {customerDeals : customerDeals, totalCount : count}});
	    		})	
    		})
    		}else{	
    			return res.status(404).json({status : 404, message : "No deal is saved yet."})
    		}
    	})
    }

    Deals.remoteMethod(
	'getCustomerDeals', 
    {
      description: 'Get all business related deals ',
      http: {path: '/customer', verb: 'get'},
      accepts: [{arg: 'customerId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'currentPage', type: 'number', 'http': {source: 'query'}},
      			{arg: 'pageSize', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });

    Deals.customerDashboard = function(customerId, res, cb){
    	if(!customerId){
    		return res.status(400).json({status : 400, message : "Customer id is required"})
    	}

    	var sql = "select count(*) from savedeals " +
				  "inner join deals on deals.id = savedeals.dealid " +
				  "where userid = "+customerId+" and deals.status = 1 "; 

		Deals.dataSource.connector.query(sql, function(err, count){
			if(err) return res.status(500).json({status : 500, message : err.message})

			return res.status(200).json({status : 200, message : "Active deals count", data: {dealsCount : count[0].count}})
		})
    } 

    Deals.remoteMethod(
	'customerDashboard', 
    {
      description: 'Get active deals count ',
      http: {path: '/customer/dashboard', verb: 'get'},
      accepts: [{arg: 'customerId', type: 'number', 'http': {source: 'query'}},
      			{arg: 'res', type: 'object', 'http': {source: 'res'}}],
      returns: {arg: 'response', type: 'string'}
    });
};
