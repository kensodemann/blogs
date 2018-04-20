# Montoring and Source Maps

TL;DR - In order for Ionic Pro Monitoring to supply source lines, the source map version and
the code version must match. For automatic source map syncing, this means the version specified
in the `config.xml` must match the version specified when calling `Pro.init()`. Details follow.

In order to get useful stacktraces from the Ionic Pro Monitoring service, you must keep your
source maps in sync with the versions of your application. To allow for multiple versions
of your code to be running on devices, Ionic Pro marks each set of source maps with the version
number that they apply to. In order for source code to be displayed in Ionic Pro Monitor, a
set of source maps with a version that exactly matches the version of the running application
must exist.

## Source Map Version

There are two ways to update the source maps. You can use the automatic method or the manual
method. Both methods require a version, but how that version is specified differs.

### Automatic Method

The automatic method is where you use `ionic monitoring syncmaps` to upload the latest source
maps. In this case, the source maps will be marked with the version that is specified in the
`widget` node of the `conifig.xml` file. For example, with the following `widget` node specified,
the source maps will be marked as being the source maps for version 1.7.3

```
<widget id="com.acme-corp.my-great-app" version="1.7.3" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
```

### Manual Method

With the manual method, you upload your source maps via the web interface. You also specify the
version number via the web interface. It is totally up to you to make sure the version specified
is correct.

## Application Version

The application version is specified when calling `Pro.init()` in your `app.module.ts` file. It
should look like this:

```
Pro.init('aaabbbcc', {
  appVersion: '1.7.3'
});
```

So long as the version specified in the `Pro.init()` call _exactly_ matches the version of an
uploaded set of source maps at the time the error is logged, you should get source information with your error logs.