# bufferingHint explained

This document proposes a new bufferingHint attribute on HTMLMediaElement and explains its usage. 

All user agents delay the start of playback until some minimum number of frames are decoded and ready for rendering. This buffer provides an important cushion against playback stalls that might otherwise be caused by intermittent decoder slowness. 

It also adds a small amount of latency to start playback. For example, in Chrome this adds roughly 200 milliseconds for common video frame rates.

# Objective

Let web authors minimize the internal buffering for use cases where sub-second latency is more important than smooth playback. At the same time, un-ship Chrome's heuristic-based approach to that triggers the minimal buffer size (the heuristics are most often wrong). 

# Use cases

A good example is cloud gaming. When a user moves their mouse to control the camera or presses a button to jump over an obstacle, even sub-second latency can make the game unplayable. A similar use case would be remote desktop streaming.

Another example is camera streaming applications. Webcams may have highly variable frame rates with occasionally very long frame durations (when nothing about the picture has changed). Buffering several long-duration frames to start playback can add considerable latency to a use case where realtime information is important.

# Discouraged uses

Most media streaming sites should prefer smooth playback over sub-second improvements in latency. This includes live streaming sports and TV, where low latency (e.g. 3 seconds) is very important, but sub-second latency is not critical and should not be prioritized over smooth playback. 

This is also central to Chrome's desire to move away from heuristic based buffering behavior. Today, Chrome uses a minimal buffer size (currently just for video rendering) whenever the container indicates that duration is unknown/infinite. While this is true of the camera streaming and cloud gaming, the vast majority of such content actually belongs this discouraged use category. 

The proposed API aims to highlight this tradeoff and avoid misuse.

# Proposed API

```Javascript
enum MediaBufferingHint { "minimal", "normal" };


partial interface HTMLMediaElement {
    attribute MediaBufferingHint bufferingHint = "normal"; 
};
```

# Example

<script>
  // Here MediaSource represents a cloud gaming stream or a stream from 
  // a secuirty camera. Not shown: MSE buffer management. 
  let mediaSource = new MediaSource();  
  video.src = window.URL.createObjectURL(mediaSource);;
  video.bufferingHint = "minimal";
</script>
