<!--
===============================================
vidgear library source-code is deployed under the Apache 2.0 License:

Copyright (c) 2019-2020 Abhishek Thakur(@abhiTronix) <abhi.una12@gmail.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
===============================================
-->

# WebGear FAQs

&nbsp;

## What is WebGear API and what does it do?

**Answer:** WebGear is as powerful **Video Streaming Server** that transfers live video-frames to any web browser on a network. _For more info. see [WebGear doc ➶](../../gears/webgear/overview/)_

&nbsp;

## How to get started with WebGear API?

**Answer:** See [WebGear doc ➶](../../gears/webgear/overview/). Still in doubt, then ask us on [Gitter ➶](https://gitter.im/vidgear/community) Community channel.

&nbsp;

## "WebGear is throwing `ModuleNotFoundError` on importing", Why?

**Answer:** This error means, VidGear is installed **WITHOUT asyncio package support** on your machine. For this support, see [Requirements ➶](../../gears/webgear/usage/#requirements).

&nbsp;

## Can WebGear always need Active Internet Connection?

**Answer:** No, it just need internet only once during [Auto-Generation Process ➶](../../gears/webgear/overview/#auto-generation-process) to download default data-files and it takes few seconds. You can also download files manually from [Github Server](https://github.com/abhiTronix/webgear_data) or even add [your own custom files](../../gears/webgear/advanced/#rules-for-altering-webgear-files-and-folders). _For more information see [Data-Files Auto-Generation WorkFlow ➶](../../gears/webgear/overview/#data-files-auto-generation-workflow-for-webgear)_

&nbsp;

## Can I manually place default files for WebGear?

**Answer:** Yes, you can either download default files from [Github Server](https://github.com/abhiTronix/webgear_data), and manually place at [default location](../../gears/webgear/overview/#default-location), OR, you can yourself create the require three critical files _(i.e `index.html`, `404.html` & `500.html`)_  inside `templates` folder at the [default location](../../gears/webgear/overview/#default-location), thereby you don't need any internet connection at all. _For more information see [Data-Files Auto-Generation WorkFlow ➶](../../gears/webgear/overview/#data-files-auto-generation-workflow)_

&nbsp;

## How can I add my custom WebPage to WebGear?

**Answer:** See [this usage example ➶](../../gears/webgear/advanced/#using-webgear-with-custom-webpage-routes).

&nbsp;

## Can I change the default location?

**Answer:** Yes, you can use WebGear's [`custom_data_location`](../../gears/webgear/params/#webgear-specific-attributes) attribute of `option` parameter in WebGear API, to change [default location](../../gears/webgear/overview/#default-location) to somewhere else.

&nbsp;

## Can I delete/rename the WebGear default data?

**Answer:** Yes, but you've to follow [these rules ➶](../../gears/webgear/advanced/#rules-for-altering-webgear-files-and-folders)

&nbsp;

## What Web browser are supported by WebGear API?

**Answer:** All modern browser with Javascript support are supported by WebGear. If not, then discuss with us on [Gitter ➶](https://gitter.im/vidgear/community) Community channel.

&nbsp;

## How to send OpenCV frames directly to Webgear Server?

**Answer:** Here's the trick to do it:

**Step-1: Trigger Auto-Generation Process:** Firstly, run any WebGear [usage example](../../gears/webgear/usage/) to trigger the [Auto-generation process](../../gears/webgear/overview/#auto-generation-process). Thereby, the default generated files will be saved at [default location](../../gears/webgear/overview/#default-location) of your machine.

**Step-2: Change Webpage address in your own HTML file:** then, Go inside `templates` directory at [default location](../../gears/webgear/overview/#default-location) of your machine, _and change the line -> `25` on `index.html` file_:

**From:**

`#!html <p class="lead"><img src="/video" class="img-fluid" alt="Feed"></p>`

**To:**

`#!html <p class="lead"><img src="/my_frames" class="img-fluid" alt="Feed"></p>`

**Step-3: Build your own Frame Producer and add it to route:** Now, create a python script code with OpenCV source, as follows:

```python
# import necessary libs
import uvicorn, asyncio, cv2
from starlette.routing import Route
from vidgear.gears.asyncio import WebGear
from vidgear.gears.asyncio.helper import reducer
from starlette.responses import StreamingResponse

# !!! define your own video source here !!!
stream = cv2.VideoCapture("/home/foo/foo.avi") 

# initialize WebGear app with same source
web = WebGear(source = "/home/foo/foo.avi", logging = True) # also enable `logging` for debugging 

# create your own frame producer
async def my_frame_producer():
  # loop over frames
  while True:
    # read frame from provided source
    (grabbed,  frame) = stream.read()
    # break if NoneType
    if not grabbed: break


    # do something with frame here


    #reducer frames size if you want more performance otherwise comment this line
    frame = reducer(frame, percentage = 50) #reduce frame by 50%
    #handle JPEG encoding
    encodedImage = cv2.imencode('.jpg', frame)[1].tobytes()
    #yield frame in byte format
    yield  (b'--frame\r\nContent-Type:image/jpeg\r\n\r\n'+encodedImage+b'\r\n')
    await asyncio.sleep(0.01)


# now create your own streaming response server
async def video_server(scope):
  assert scope['type'] == 'http'
  return StreamingResponse(my_frame_producer(), media_type='multipart/x-mixed-replace; boundary=frame') # add your frame producer



# append new route to point your own streaming response server created above
web.routes.append(Route('/my_frames', endpoint=video_server)) #new route for your frames producer will be `{address}/my_frames`

# run this app on Uvicorn server at address http://localhost:8000/
uvicorn.run(web(), host='localhost', port=8000)

# close app safely
web.shutdown()
```

**Final Step:** Finally, you can run the above python script, and see the desire output at address http://localhost:8000/ on your browser. 

&nbsp;