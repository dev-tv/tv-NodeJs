var timediff = require('timediff');
var NodeGeocoder = require('node-geocoder');
var moment = require('moment-timezone'); 
var config = require('../server/config.json');
var _ = require('underscore');
var app = require('../server/server.js');


function formatedDateTime(timestamp) {
	timestamp = parseInt(timestamp);
	var to = Date.now();
	var time = timediff(timestamp, to, 'YMDHmmS');

	if (time.hours != 0 && time.days == 0) {
	if (time.hours == 1) {
	returnvalue = time.hours + "h ago";
	} else {
	returnvalue = time.hours + "h ago";
	}
	} else if (time.minutes != 0 && time.days == 0) {
	if (time.minutes == 1) {
	returnvalue = time.minutes + "m ago";
	} else {
	returnvalue = time.minutes + "m ago";
	}
	} else if (time.seconds != 0 && time.days == 0) {
	if (time.seconds == 1) {
	returnvalue = time.seconds + "s ago";
	} else {
	returnvalue = time.seconds + "s ago";
	}
	}else if (time.seconds == 0 && time.days == 0) {
	returnvalue = "Now";
	}else{
	var date = new Date(timestamp);
	var returnvalue = ('0' + (date.getMonth() + 1)).slice(-2) + '/' +('0' + date.getDate()).slice(-2) + '/' + date.getFullYear();
	}
	return returnvalue;
}

function formatedDateTime1(timestamp) {
	timestamp = parseInt(timestamp);
	var to = Date.now();
	var time = timediff(timestamp, to, 'YMDHmmS');
	var returnvalue = "";
	
	if (time.days == 0) {
		returnvalue = "Today's Sighting"
	}else{
		var date = new Date(timestamp);
		returnvalue = ('0' + (date.getMonth() + 1)).slice(-2) + '/' +('0' + date.getDate()).slice(-2) + '/' + date.getFullYear();
	}
	return returnvalue;
}

function getCallerIP (request) {
	var ip = request.headers['x-forwarded-for'] ||
	request.connection.remoteAddress ||
	request.socket.remoteAddress ||
	request.connection.socket.remoteAddress;

	ip = ip.split(',')[0];
	ip = ip.split(':').slice(-1); 
	return ip[0];
}

function checkOneDayAgoData(date){
	var dateDiff = Date.now() - date;
	var hours = Math.floor(dateDiff / (60 * 60 * 1000));	
	if(hours <= 24){
		return true;
	}else{
		return false;
	}
}

function generateSlug(name){
	var slug = name.replace(/\s+/g, '-').toLowerCase();
	return slug; 
}


function getFirstDateOfMonth(){
	var date = new Date();
	var firstDay = moment(new Date(date.getFullYear(), date.getMonth(), 1)).format("YYYY-MM-DD");
	var lastDay = moment(new Date(date.getFullYear(), date.getMonth() + 1, 0)).format("YYYY-MM-DD");

	return {
		firstDay : firstDay,
		lastDay : lastDay
	}
}

function getFirstDateOfWeek(){
	var curr = new Date; 
	var first = curr.getDate() - curr.getDay(); 
	var last = first + 6; 

	var firstday = moment(new Date(curr.setDate(first))).add(1, 'days').format("YYYY-MM-DD");
  	var lastday = moment(new Date(curr.setDate(last))).add(1, 'days').format("YYYY-MM-DD");

	return {
		firstDay : firstday,
		lastDay : lastday
	}
}
// it converts the 1000 to 1K and 1000000 to 1M and so on ......
function nFormatter(num) {
     if (num >= 1000000000) {
        return (num / 1000000000).toFixed(1).replace(/\.0$/, '') + 'B';
     }
     if (num >= 1000000) {
        return (num / 1000000).toFixed(1).replace(/\.0$/, '') + 'M';
     }
     if (num >= 1000) {
        return (num / 1000).toFixed(1).replace(/\.0$/, '') + 'K';
     }
     return num;
}


function compareDays(wrongSquence) {
   var outputFormat = ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"];
   var newArray = [];
   var i=0;
   for(i=0;i<outputFormat.length;i++) {
      var obj = _.find(wrongSquence, function(obj){ return obj.day == outputFormat[i]; });
      newArray.push(obj);
   } 
   return newArray;
}

module.exports = {
  formatedDateTime : formatedDateTime,
  formatedDateTime1 : formatedDateTime1,
  getCallerIP : getCallerIP,
  checkOneDayAgoData : checkOneDayAgoData,
  getFirstDateOfMonth : getFirstDateOfMonth,
  getFirstDateOfWeek : getFirstDateOfWeek,
  nFormatter : nFormatter,
  generateSlug : generateSlug,
  compareDays : compareDays
};
