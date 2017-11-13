# oklog-ui

OK Log UI is a Vue.js app to inspect and filter OK Log entries stored in json format.

It will display non-json formatted log entries, but there's not much processing available to them.

The real power comes when your logs are stored in json format. You can use [jq](https://stedolan.github.io/jq/) filters to do advanced processing on your logs.

## Hosting

You can host this app yourself, or checkout a demo at https://oklog-ui.firebaseapp.com/

UI uses IndexedDB storage local to your browser, so you can use the above app and not host locally, if you'd like!

### Building

If you want to build and host this app yourself, just `yarn run build` and then upload the `dist/` folder to your hosting provider.

## OK Log Server

UI accesses your OK Log server via javascript, so you'll need to have [this](https://github.com/oklog/oklog/pull/80) patch applied to your api until the next version of OK Log is released.
