# proton-pack

Cross the streams.

## Summary

Serverless deployment of a video processing pipeline for capturing snapshots/clips from `N` concurrent video streams based on an event within a single stream.

## Use Cases

* Capture multiple angles when an intruder enters the doorway
* Capture emotion and target of their des(ire)
* Capture scene from multiple focal lengths
* Capture non-repeatable events from multiple angles, e.g. stunts

## Technology

* [Kinesis Data Streams](https://aws.amazon.com/kinesis/)
* [Kinesis Video Streams](https://aws.amazon.com/kinesis/video-streams/)
* [Rekognition](https://aws.amazon.com/rekognition/)
* [DynamoDB](https://aws.amazon.com/dynamodb/)
* [S3](https://aws.amazon.com/s3/)
* [Lambda](https://aws.amazon.com/lambda/)
* [Terraform](https://www.terraform.io/)

## Architecture Diagram

TODO

## Data sources

Two or more cameras that can be connected to the internet.

## Challenges

* Camera: Properly streaming `h.264` from multiple cameras, e.g. GoPros
* Coordination: The accuracy of frame capture between multiple video streams is paramount. If the snapshots are off by too much of a delta, even a second, they won't line up.
* Scaling: The combination of camera framerate and pipeline sampling speed will push the number of parallel lambda functions
* Cost: Doing this "on the cheap". We aren't going to sample and index every frame.

## Product

Before we break this down into MVP and beyond, let's look at what the end product looks like.

* Streaming
  * Setup local environment for streaming video from cameras into Kinesis Video
* Extraction
  * Extract frames from video streams as they come in
  * Using API + Lambda or Rekognition processor as input
  * Write frame to S3 and frame metadata to DynamoDB
* Event Detection
  * Trigger off S3 PUT or DynamoDB stream
  * Detect event based on some configurable criteria (emotion, specific person, object in scene)
  * If no event, delete frame in S3 and frame metadata in DynamoDB
* Frame Extraction
  * Trigger off result of Event Detection; run one for each video stream
  * Pull frame from S3 and frame metadata from DynamoDB
  * Pull corresponding frame from other video streams
  * Write frames to S3 and metadata to DynamoDB
* Frame Joiner
  * Trigger (one) off result of Frame Extraction
  * Pull all frames from S3 and frame metadata from DynamoDB
  * Merge frames into single image (PnP)
  * Write frame to S3 and metadata to DynamoDB
* UX
  * Site that allows browsing of joined images

### MVP

* Two web cameras
* Event detection is simple time interval, say every 5 seconds
* No Frame Joiner
* UX displays all raw frames

### Beyond

* Additional cameras, e.g. Go Pro
* Event detection (emotion, object in scene, etc.)
* Clip Extraction (defined duration)
* Frame Joiner merges frames from all snapshots
* UX is browser for all images e.g. carousel
