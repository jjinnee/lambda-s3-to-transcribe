# Title

![image](https://user-images.githubusercontent.com/46839654/150350374-aac71b2c-44ce-4ab1-9588-7fab916b8f3b.png)

    const AWS = require("aws-sdk");
    const transcribe = new AWS.TranscribeService();

    const LANGUAGE_CODE = "en-US";

    exports.handler = async (event, context) => {
        const eventRecord = event.Records && event.Records[0];
        const inputBucket = eventRecord.s3.bucket.name;
        const key = eventRecord.s3.object.key;
        const id = context.awsRequestId;
        const ext = `.${key.split(".")[1]}`;

        const fileUri = `s3://${inputBucket}/${key}`;
        const jobName = `${key}-${id}`;

        const config = {
            TranscriptionJobName: jobName,
            LanguageCode: LANGUAGE_CODE,
            Media: {
                MediaFileUri: fileUri
            },
            Subtitles: {
                Formats: ["srt", "vtt"]
            }
        }

        return transcribe.startTranscriptionJob(config).promise();
    };
    
    
[startTranscriptionJob docs](https://docs.aws.amazon.com/transcribe/latest/dg/API_StartTranscriptionJob.html)

### Create & delete jobs

    const AWS = require("aws-sdk");
    const transcribe = new AWS.TranscribeService();

    const LANGUAGE_CODE = "en-US";

    exports.handler = async (event, context) => {
        const eventRecord = event.Records && event.Records[0];
        const eventName = eventRecord.eventName;

        if (eventName === "ObjectCreated:Put") {
            const inputBucket = eventRecord.s3.bucket.name;
            const key = eventRecord.s3.object.key;
            const id = context.awsRequestId;
            const ext = `.${key.split(".")[1]}`;

            const fileUri = `s3://${inputBucket}/${key}`;
            const jobName = `${key}`;

            const config = {
                TranscriptionJobName: jobName,
                LanguageCode: LANGUAGE_CODE,
                Media: {
                    MediaFileUri: fileUri
                },
                Subtitles: {
                    Formats: ["srt", "vtt"]
                }
            };

            return transcribe.startTranscriptionJob(config).promise();
        } else if (eventName === "ObjectRemoved:Delete") {
            const key = eventRecord.s3.object.key;
            const config = {
                TranscriptionJobName: key
            }

            return transcribe.deleteTranscriptionJob(config).promise();
        }

        return 0;
    };
