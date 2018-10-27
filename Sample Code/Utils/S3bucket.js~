var AWS = require('aws-sdk');
var config = require('../server/config.json');

AWS.config.update({
    accessKeyId: config.s3bucket.accessKey,
    secretAccessKey: config.s3bucket.secretKey,
    region: config.s3bucket.region
});


var s3 = new AWS.S3({ params: {Bucket: config.s3bucket.bucketName} });

exports.generateSignedUrl = function(imageName, contentType, callback){

    var params = {Key: imageName,ContentType: contentType};
    s3.getSignedUrl('putObject', params, function (err, url) {
        if(err) {
            callback(err , null);
        }
         callback(null, url);
    });
}
