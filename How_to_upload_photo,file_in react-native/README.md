# How to upload photo/file in react-native

If you wanted to upload photo/file in react-native, almostly you have found many library such as [react-native-uploader](https://github.com/aroth/react-native-uploader) , [react-native-file-upload](https://github.com/booxood/react-native-file-upload). But I think easiest way is this post, not use library, not write native code, and anybody can understand. 

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



# Sample

use react-native-image-picker and react-native-camera-roll-picker(based on React Native CameraRoll API). 

## [Here](https://github.com/g6ling/react-native-fileUpload-example)

