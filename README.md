
# Smart Downloader

## Description
HTTP(s) files downloader that can resume your downloads and supports download speed limits (throttle).
It also supports file download progress callback to receive progress value in percentage.

Script ensures (creates if missing) that destination dir exists.

You can also extract downloaded archive file, just specify where to.
It supports tarball (gzipped as well) and zip files.

If you already know md5 checksum value for the file to be downloaded, you can also validate downloaded file against it.

*Important - it currently uses `wget` to make file downloads resumable and throttled, so it might not work on all operating systems.
There is a plan to add a fall back to just normal download for unsupported operating systems.

## Install
```npm install --save smart-downloader```

## Simple usage
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader();

downloader.download({
    uri: 'https://github.com/gegis/smart-downloader/raw/master/test/fixtures/code-1.jpg',
    destinationDir: './downloads/'
}, (err, data) => {

    if (err) {

        console.log(err);
    }

    console.log(data);
});
```

## Advanced usage
A file download with an option to resume and limit download speed to 125 kBps (125 KiloBytes = 1 Megabit) and a custom destination file name:
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader();

downloader.download({
    uri: 'https://github.com/gegis/smart-downloader/raw/master/test/fixtures/code-1.jpg',
    destinationDir: './downloads/',
    destinationFileName: 'advanced-1.jpg',
    resumeDownload: true,
    downloadSpeedLimit: 125 //value in KiloBytes per second
}, (err, data) => {

    if (err) {

        console.log(err);
    }

    console.log(data);
});
```
You can also specify defaults in constructor and specify md5 checksum value for the downloaded file to be verified:
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader({
    resumeDownload: true,
    downloadSpeedLimit: 25 //value in KiloBytes per second
});

downloader.download({
    uri: 'https://github.com/gegis/smart-downloader/raw/master/test/fixtures/code-1.jpg',
    destinationDir: './downloads/',
    destinationFileName: 'advanced-2.jpg',
    md5: 'd4a63031f57bdcafb86ca02100fdd6d2'
}, (err, data) => {

    if (err) {

        console.log(err);
    }

    console.log(data);
});
```
You can pass progress callback to get download progress updates:
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader({
    resumeDownload: true,
    downloadSpeedLimit: 250, //value in KiloBytes per second
    progressUpdateInterval: 2000 // value in milliseconds
});

downloader.download({
    uri: 'https://unsplash.com/photos/cvBBO4PzWPg/download?force=true',
    destinationDir: './downloads/',
    destinationFileName: 'advanced-3.jpg',
    md5: 'd94f347a514c051cff5a28814ddacb73'
}, (err, data) => {

    if (err) {

        console.log(err);
    }
    console.log(data);
}, (err, data) => {

    if (err) {

        console.log(err);
    }
    console.log(`Progress: ${data.progress}%`);
});
```
You can pass extract dir to extract (unzip) archived file contents:
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader();

downloader.download({
    uri: 'https://github.com/gegis/smart-downloader/raw/master/test/fixtures/code.tar.gz',
    destinationDir: './downloads/',
    extractDir: './downloads/code'
}, (err, data) => {

    if (err) {

        console.log(err);
    }
    console.log(data);
});
```

Specify request Headers (it accepts the array of header strings):
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader();

downloader.download({
    uri: 'https://github.com/gegis/smart-downloader/raw/master/test/fixtures/code.tar.gz',
    destinationDir: './downloads/',
    extractDir: './downloads/code',
    headers: ['Accept-Language: "en-us"', "Accept-Encoding: 'gzip, deflate'"]
}, (err, data) => {

    if (err) {

        console.log(err);
    }
    console.log(data);
});
```

Specify other wget related options (it accepts the array of options):
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader();

downloader.download({
    uri: 'https://github.com/gegis/smart-downloader/raw/master/test/fixtures/code.tar.gz',
    destinationDir: './downloads/',
    extractDir: './downloads/code',
    wgetOptions: ['--no-dns-cache', '--wait=1']
}, (err, data) => {

    if (err) {

        console.log(err);
    }
    console.log(data);
});
```

Example of how to stop ("pause") current download process:
```
const SmartDownloader = require('smart-downloader');

const downloader = new SmartDownloader();

// It returns child process instance
const cmd = downloader.download({
    uri: 'https://unsplash.com/photos/cvBBO4PzWPg/download?force=true',
    destinationDir: './downloads/',
    destinationFileName: 'test-stop.jpg,
    downloadSpeedLimit: 300
}, (err, data) => {

    if (err) {

        // Handling the known error
        if (err.message === 'Download error. Signal: SIGTERM') {

            console.log('Handled stop);
        } else {

            console.log(err);
        }
    }
    console.log(data);
});

// After 3 seconds we decide we want to stop it from downloading
setTimeout(() => {

    // You can kill it with custom signal, i.e. "SIGINT"
    // By default it's SIGTERM signal
    // It will cause an error in your callback, make sure to handle it
    cmd.kill();
}, 3000);
```

## Options
Can be passed either to constructor or `download` function:
- `uri` - url to download file from
- `destinationDir` - destination directory, absolute or relative path
- `resumeDownload` - resume download, default: `true`
- `downloadSpeedLimit` - download speed limit integer value, default: `null`
- `downloadSpeedLimitUnit` - download speed limit unit value, default `'k'` (KiloBytes)
- `progressUpdateInterval` - it tells how often to update with download progress, value in ms, default `1000`
- `destinationFileName` - downloaded file name in destination dir, otherwise it will use original file name
- `md5` - if md5 checksum specified, it will verify downloaded file md5 checksum against it
- `extractDir` - if specified, it will extract downloaded archive to the specified dir
- `headers` - (array) if specified, it will append headers to request
- `wgetOptions` - (array) if specified, it will append all options related to wget command
