# Using Lambda to Create Filterable S3 Event Notifications for SNS

This creates an [AWS Lambda](https://aws.amazon.com/lambda/) for adding the bucket, key, and filename as message attributes onto SNS notifications. This allows for filtering of the event notifications on the subscription side using [message filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html).

Currently, S3 Event Notifications can be filter for [specific prefixes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-how-to-filtering.html) but only when setting up the notifications from S3 to SNS. This means that for public dataset buckets with N different prefixes would require N different public SNS topics with the users having to subscribe to the correct topics.

This Lambda function adds `bucket`, `key`, and `filename` as `String` message attributes onto the SNS notifications. The filename is determined using the `os.path.basename` Python function and thus subject to any existing limitations around its ability to parse the key as a path. 

## Setting up this Lambda Function

The provided CloudFormation template `cloudformation.yaml` will create the Lambda function, subscribe to an existing public SNS topic fed by S3 Event Notifications, and create a new public SNS topic with filterable S3 Event Notifications.


## Creating Filters in CloudFormation

To add filters to your existing cloudformation topic subscriptions you can use the [`FilterPolicy` attribute](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-subscription.html#cfn-sns-subscription-filterpolicy).

As an example if you only wish to receive the GOES-16 ABI L2 Channel 1 over the full disk notifications you would write a filter policy such as the following based upon the file names:

```
    FilterPolicy:
        filename:
            - prefix: "OR_ABI-L2-CMIPF-M6C01"
            - prefix: "OR_ABI-L2-CMIPF-M5C01"
            - prefix: "OR_ABI-L2-CMIPF-M3C01"
            - prefix: "OR_ABI-L2-CMIPF-M4C01"
```

To only receive notifications for new ABI L2 products from any channel over the full disk you can write a prefix filter such as:

```
    FilterPolicy:
        key:
            - prefix: "ABI-L2-CMIPF"
```

## Pricing

The Lambda function defaults to 128mb of memory when built with the CloudFormation template. Based on running this against one dataset for a few days, the p99 and p50 Lambda execution durations are 375ms and 197ms respectively. Using [calculator.aws](https://calculator.aws/#/createCalculator/Lambda) we can estimate the monthly pricing for running this. The [NEXRAD Level II real-time data](https://registry.opendata.aws/noaa-nexrad/) open dataset sees about 3 million new object notifications a day, or 90 million per month. Plugging this into the calculator without free tier and using the p99 duration, we estimate the maximum cost for running this to be $89/month/dataset. 

