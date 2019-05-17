# renderingBufferHint explained

This document proposes a new renderingBufferHint attribute on HTMLMediaElement and explains its usage. 

All user agents delay the start of playback until some minimum number of frames are decoded and ready for rendering. This buffer provides an important cushion against playback stalls that might otherwise be caused by intermittent decoder slowness. 

It also adds a small amount of latency to start playback. For example, in Chromium this adds roughly 200 milliseconds for common video frame rates.

# Objective

Let web authors disable the internal rendering buffer for use cases where sub-second latency is more important than smooth playback. At the same time, un-ship Chromium's heuristic-based approach that disables the rendering buffer (the heuristics are most often wrong). 

# Use cases

A good example is cloud gaming. When a user moves their mouse to control the camera or presses a button to jump over an obstacle, even sub-second latency can make the game unplayable. A similar use case would be remote desktop streaming.

Another example is camera streaming applications. Webcams may have highly variable frame rates with occasionally very long frame durations (when nothing about the picture has changed). Buffering several long-duration frames to start playback can add considerable latency to a use case where realtime information is important.

# Discouraged uses

Most media streaming sites should prefer smooth playback over sub-second improvements in latency. This includes live streaming sports and TV, where low latency (e.g. 3 seconds) is very important, but sub-second latency is not critical and should not be prioritized over smooth playback. 

This is also central to Chromium's desire to move away from heuristic based buffering behavior. Today, Chromium disables the buffer (currently just for video rendering) whenever the media metadata indicates that duration is unknown/infinite. While this is true of the camera streaming and cloud gaming, the vast majority of such content actually belongs this discouraged use category. 

The proposed API aims to highlight this tradeoff and avoid misuse.

# Proposed API

```Javascript
enum RenderingBufferHint { "none", "normal" };


partial interface HTMLMediaElement {
    attribute RenderingBufferHint renderingBufferHint = "normal"; 
};
```

Callers who change the value to "none" are encouraged to set the attribute before triggering [the media load algorithm](https://html.spec.whatwg.org/multipage/media.html#media-element-load-algorithm) (e.g. before setting the src attribute). This maximizes the player's opportunity to apply the setting to parts of the stack that cannot easily be changed once loading begins (e.g. OS audio buffer output size).

# Example
```Javascript
<script>
  // Here MediaSource represents a cloud gaming stream or a stream from 
  // a secuirty camera. Not shown: MSE buffer management. 
  let mediaSource = new MediaSource();  
  video.renderingBufferHint = "none";
  video.src = window.URL.createObjectURL(mediaSource);
</script>
```
