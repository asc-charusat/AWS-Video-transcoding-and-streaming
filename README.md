# AWS-Video-transcoding-and-streaming
The Pandemic crossed the lengths when people weren’t allowed to go out, but the only way it all turned out well, was because of the Online streaming. From a small gathering to worship god, singing holy songs on live streams, to getting entertained, streaming helped everyone. Especially at this time, new emerging platforms for streaming, transcoding and giving out an optimized result, on several devices like smartphones, laptops, tablets, etc. AWS, and its several services like Elastic Transcoder, S3 buckets, CloudFront tops the chart. Streaming with AWS can be entertaining, fun, and secure as well. This project aims to find out, the optimization, delivering the streamed videos with minimal latency.

var AWS = require('aws-sdk');
var s3 = new AWS.S3({
  apiVersion: '2012–09–25'
});
var eltr = new AWS.ElasticTranscoder({
  apiVersion: '2012–09–25',
  region: 'us-east-1'
});
exports.handler = function(event, context) {
  var bucket = event.Records[0].s3.bucket.name;
  var key = event.Records[0].s3.object.key;
  var pipelineId = '1519927986763-h63mfd';
  
  if (bucket !== 'streaming-video-raw') {
    context.fail('Incorrect input bucket');
    return;
  }
  
  var srcKey = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " ")); // the object may have spaces  
  var newKey = key.split('.')[0];
  var params = {
    PipelineId: pipelineId,
    OutputKeyPrefix: newKey + '/',
    Input: {
      Key: srcKey,
      FrameRate: 'auto',
      Resolution: 'auto',
      AspectRatio: 'auto',
      Interlaced: 'auto',
      Container: 'auto'
    },
    Outputs: [{
      Key: newKey + '.mp4',
      ThumbnailPattern: 'thumbs-' + newKey + '-{count}',
      PresetId: '1351620000001-100020',
      Watermarks: [],
    }]
  };
 
  console.log('Starting a job');
  eltr.createJob(params, function(err, data) {
    if (err){
      console.log(err);
    } else {
      console.log(data);
    }
    context.succeed('Job successfully completed');
  });
};
When you paste the above code to your Lambda function editor, replace the pipelineId variable with the with ID of the pipeline in your account. You can get the pipeline ID by using the AWS terminal command like so:
aws elastictranscoder list-pipelines
The output JSON string will contain the pipeline Id field. Paste its value into your AWS Lambda function. The other variable you might want to change in the future is the PresetId . The value the above code sample contains refers to a preset that transcodes the video to a progressively streaming MP4 file optimized for smartphones and tablets. If AWS changes the Id of the preset in the future, you might need to get the new value. You can retrieve it by running this command:
aws elastictranscoder list-presets
Scroll through the JSON output and find the Id of the preset called “System preset: iPhone 4s and above, iPad 3G and above, iPad mini, Samsung Galaxy S2/S3/Tab 2”. Set the value of the PresetId variable in our function code and click on “Create function” to submit the form. Your function is now ready.
