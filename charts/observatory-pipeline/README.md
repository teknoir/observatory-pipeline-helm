# Observatory Pipeline Helm Chart

This chart deploys Observatory Pipelines to a Kubernetes cluster.

> The implementation of the Helm chart is right now the bare minimum to get it to work.

## Usage in Teknoir platform
Use the HelmChart to deploy the Observatory Pipeline to a Device.

```yaml
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: observatory-pipeline
  namespace: default
spec:
  repo: https://teknoir.github.io/observatory-pipeline-helm
  chart: observatory-pipeline
  targetNamespace: default
  valuesContent: |-
    # Example for minimal configuration
    streams:
      - stream: update-frame-rate-control
    
    # Values possible for Observatory Pipelines
    defaults:
      stream: update-frame-rate-control
      image:
        repository: us-docker.pkg.dev/teknoir/gcr.io/observatory-event-processing
        tag: dev-4552014
      # Set the log level for the nvr
      # - DEBUG
      # - INFO
      # - WARNING
      # - ERROR
      debugLevel: INFO
      mqtt:
        # Local MQTT broker
        broker: mqtt.kube-system
        port: 1883
        username: "emqx_operator_controller"
        password: ""
        qos: 2
        keepAliveInterval: 10
        connectRetryInterval: 10
        # Topics
        topicSink: "events/sink"
        topicSource: "events/source"
        topicAllCameras: "camera/frames/#"
        topicDetections: "events/detections"
        topicInference: "events/inference"
        topicMovements: "events/movements"
        topicFilteredMovements: "events/filtered-movements"
        topicAlerts: "events/alerts"
        topicStaleAlerts: "events/stale-alerts"
        topicSensors: "events/sensors"
        topicMeasurements: "events/measurements"
        topicToeEvents: "toe/events"
        topicToeCommands: "toe/commands/devstudio"
        topicAllDevices: "/devices/+/events"
      resources:
        limits:
          cpu: 2000m
          memory: 2048Mi
        requests:
          cpu: 100m
          memory: 128Mi
      fe:
        frontEndDefaultInterval: 60 # seconds
      historian:
        apiUrl: "http://historian-api:8000"
        recordDetectionsInterval: 333 # milliseconds
        objectsToFilter: "car,truck,motorbike,person,face" # supports multiple labels via comma separated list
      regions:
        iouThreshold: 0.001 # default Intersection Over Union (IOU) threshold for object detection inside region
      video:
        minStartBeforeAlert: 10 # seconds
        mountSegmentPath: false
        segmentPath: "/opt/teknoir/video/segments"
        mountClipTempPath: false
        clipTempPath: "/opt/teknoir/video/clips"
        segmentLength: 15 # seconds
        segmentRetentionLength: 5760 # segments
        maxClipLength: 600 # seconds
        clipRunupLength: 10 # seconds
        chunkMaxSize: 500000 # bytes
        chunkRetentionSeconds: 300 # seconds
        videoBasePath: "media/videos"
        hasAnnotations: true
        annotationsBucketPath: "media/annotations"
        snapshotBucketPath: "media/snapshots"
        staleSeconds: 30 # seconds
      alerts:
        personDownInterval: 10
        personDownScoreThreshold: 0.85
        personUpScoreThreshold: 0.75
        personDownUpTotalCountThreshold: 10
        loiteringLabels: "person" # supports multiple labels via comma separated list
        sensorsNodeData: "{}"
        loiteringThreshold: 60 # 60 seconds
        loiteringCheckStationary: true
        loiteringStationaryThreshold: 0.01 # 1% of FOV
        repetitiveAlertsCheck: true
        repetitiveAlertsTimespan: 3600 # seconds
        repetitiveAlertsIouThreshold: 0.9 # 90%
        safeOpenWindowSizeSeconds: 5 # Default window size of 5 seconds
        numberOfDominantLabels: 5
        motionAlertMinInterval: 30 # 30 seconds
```

## Example values.yaml

```yaml
defaults:
  debugLevel: DEBUG

streams:
  - stream: update-frame-rate-control
    mqtt:
      topicDetections: "camera/frames/#"
  - stream: record-detections
    mqtt:
      topicDetections: "camera/frames/#"
  - stream: update-movements
  - stream: filter-movements
  # ALERTS
  - stream: motion
  - stream: loitering
  - stream: person-down
  - stream: catalytic-theft
  # ALERT PROCESSING
  - stream: update-alerts
  - stream: make-clip
    video:
      mountSegmentPath: true
      mountClipTempPath: true
```

## Adding the repository

```bash
helm repo add teknoir-observatory-pipeline https://teknoir.github.io/observatory-pipeline-helm/
```

## Installing the chart

```bash
helm install observatory-pipeline teknoir-observatory-pipeline/observatory-pipeline -f values.yaml
```