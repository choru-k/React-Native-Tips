# How to upload photo/file in react-native

If you wanted to upload photo/file in react-native, almostly you have found many library such as [react-native-uploader](https://github.com/aroth/react-native-uploader) , [react-native-file-upload](https://github.com/booxood/react-native-file-upload). But I think easiest way is this post, not use library, not write native code, and anybody can understand. 

# Index
- Use fetch
- How can I get photo from gallery?
- How can I add progress?
- Sample (server, photoGallery, imageUpload)
- Add progress using fetch
- Use other network libraries

# use fetch 

Fetch supports `multipart/form-data`. You can upload your `formdata` like in webbrower. see [this](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#Body)

### example



```jsx
const data = new FormData();
data.append('name', 'testName'); // you can append anyone.
data.append('photo', {
  uri: photo.uri,
  type: 'image/jpeg', // or photo.type
  name: 'testPhotoName'
});
fetch(url, {
  method: 'post',
  body: data
}).then(res => {
  console.log(res)
});
```

only this. If you want many photos

```js
const photos = [photo1, photo2, ...]
photos.forEach((photo) => {
    data.append('photo', {
    uri: photo.uri,
    type: 'image/jpeg', // or photo.type
    name: photos.name
  });  
});
fetch(url, opts);

```



# How can I get photo from gallery?

You can use [React Native API](https://facebook.github.io/react-native/docs/cameraroll.html), or other library such as [react-native-image-picker](https://github.com/marcshilling/react-native-image-picker),  [react-native-camera-roll-picker](https://github.com/jeanpan/react-native-camera-roll-picker). 

#### Tips:

In iOS+10, you must add 

```xml
 <key>NSPhotoLibraryUsageDescription</key>
 <string> Why you want Photo? </string>
```

In Android, you must add

```xml
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-feature android:name="android.hardware.camera" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>

```



# How can I add progress?

Unfortunately, fetch doen't support progress. But we can use xhr which support progress.

I make `api.js`

```javascript
const futch = (url, opts={}, onProgress) => {
    console.log(url, opts)
    return new Promise( (res, rej)=>{
        var xhr = new XMLHttpRequest();
        xhr.open(opts.method || 'get', url);
        for (var k in opts.headers||{})
            xhr.setRequestHeader(k, opts.headers[k]);
        xhr.onload = e => res(e.target);
        xhr.onerror = rej;
        if (xhr.upload && onProgress)
            xhr.upload.onprogress = onProgress; // event.loaded / event.total * 100 ; //event.lengthComputable
        xhr.send(opts.body);
    });
}
export default futch

```

Use this.

### example

```javascript
import futch from './api';

const data = new FormData();
data.append('name', 'testName');
data.append('photo', {
  uri: source.uri,
  type: 'image/jpeg',
  name: 'testPhotoName'
});


futch(url, {
  method: 'post',
  body: data
}, (progressEvent) => {
  const progress = progressEvent.loaded / progressEvent.total;
  console.log(progress);
}).then((res) => console.log(res), (err) => console.log(err))
```



# Sample (server, photoGallery, imageUpload)

This sample uses react-native-image-picker and react-native-camera-roll-picker(based on React Native CameraRoll API). 
Server uses nodeJS.

## [Here](https://github.com/g6ling/react-native-fileUpload-example)

# Add Progress with fetch
I think this way using two ways which are fetch and futch in networking is not pretty.
So I overwrite fetch. 

```javascript
import futch from './src/api';
const originalFetch = fetch
global.fetch = (url, opts) => {
  console.log(opts.onProgress)
  if (opts.onProgress && typeof opts.onProgress === 'function') {
    return futch(url, opts, opts.onProgress)
  } return originalFetch(url, opts)
}
export default class photoUploadTest extends Component {
 ...
}
```
If you add this in your top file like `index.ios.js`, you can use fetch with progress.

```javascript
fetch(url + '/array', {
      method: 'post',
      body: data,
      onProgress: (e) => {
        const progress = e.loaded / e.total;
        console.log(progress);
        this.setState({
          progress: progress
        });
      }
    }).then((res) => console.log(res), (e) => console.log(e))
```



# Using other libraries

If you use another library such as axios, apisauce(wrapper axios), you can use progress config.

### example

In this case, I use [apisauce](https://github.com/skellock/apisauce). Apisauce with [reactotron](https://github.com/reactotron/reactotron) is awesome. If you don't know this, try.

```js
import { create } from 'apisauce'

// create api. 
const api = create({
  baseURL: 'http://localhost:3000',
})

// create formdata
const data = new FormData();
    data.append('name', 'testName');
    photos.forEach((photo, index) => {
      data.append('photos', {
        uri: photo.uri,
        type: 'image/jpeg',
        name: 'image'+index
      });
    });

// post your data.
api.post('/array', data, {
      onUploadProgress: (e) => {
        console.log(e)
        const progress = e.loaded / e.total;
        console.log(progress);
        this.setState({
          progress: progress
        });
      }
    })
      .then((res) => console.log(res))

// if you want to add DonwloadProgress, use onDownloadProgress
onDownloadProgress: (e) => {
  const progress = e.loaded / e.total;
}

```



This way is best I think.

