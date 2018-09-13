# on-premise
On premise distribution of Upscope

## Installation and quick start
Requirements:
- MongoDB
- Redis

1. Download Upscope from https://dist.upscope.io/index.html with the username and password provided
2. Run the server by running `BYPASS_AUTH=yes ./upscope-data-[macos|linux|windows.exe]` depending on your platform
3. Add the following code to any HTML page:
```
(function(w, u, d){if(typeof u!=="function"){var i=function(){i.c(arguments)};i.q=[];i.c=function(args){i.q.push(args)};
w.Upscope=i;var l = function(){var s=d.createElement('script');s.type='text/javascript';s.async=true;
s.src='http://localhost:5002/upscope.js';var x=d.getElementsByTagName('script')[0];x.parentNode.insertBefore(s,x);};l();}}
)(window, window.Upscope, document);

Upscope('init', {
apiKey: 'test'
});

Upscope('getWatchLink', console.log);
```

## Authenticating API requests
To perform a REST request to the API, you'll need to set the `Authorization` header to `Bearer API_KEY`, with `API_KEY` being
the `REST_KEY` set as an environment variable (in production) or `upscope123` (in development).

## Authenticating the agent
Unless `BYPASS_AUTH` is set to `yes`, agents will be redirect to `AUTH_ENDPOINT` with the ID of the visitor they intend
to screen share with as a query parameter.

Your server will need to authenticate the agent and use the following endpoint to generate a signed url to initiate
screen sharing.

```
POST BASE_ENDPOINT/api/users/:visitor_id/watch_url
{
  "agent": {
    "id": "unique_agent_identifier",
    "name": "Name of the agent"
  }
}
```

A successful request will return a JSON response like this:
```
{
  "status": "ok",
  "url": "http://localhost:5002/screen?id=123&token=abc.def"
}
```
Redirecting the agent to `url` will initiate the screen share.

## Retrieving information about users
You can retrieve information about a user by making a `GET` request to the following endpoint:
```
GET BASE_ENDPOINT/api/users/:visitor_id
```

You can retrieve a list of the most recent users:
```
GET BASE_ENDPOINT/api/users/list/latest?apiKey=abc
```
with `apiKey` being the key used as api_key in the javascript configuration for that user.

You can retrieve information about a user by making a `GET` request to the following endpoint:
```
GET BASE_ENDPOINT/api/users/list/search?q=search_query&apiKey=abc
```
with `apiKey` being the key used as api_key in the javascript configuration for that user, and `q` being the search query.

## Tracking usage
To correctly bill you for your on-premise usage, you will need to report usage to Upscope. If you are able to keep outgoing
connections open, this will happen automatically and we will record usage for you by making requests to our service.

These connections do not include any user data whatsoever.

### Tracking usage manually
By setting `REPORT_USAGE_ENDPOINT` to a custom URL you are able to intercept our outoing usage calls and track usage for us.
You'll then be able to report usage to us as agreed with your implementation manager.

Upscope will make `POST` requests to the endpoint at the start, end and every 60 seconds during each screen share. The
content will be a json encoded document which looks like this:
```
{
  apiKey: "api key of the visitor",
  shortId: "short id of the visitor",
  seconds: 60, // number of seconds since last report
  ended: false, // true if screen share has ended
  features_used: [], // Array of features used
  agent_id: "id of the agent",
  went_live: true // Set to true if screen sharing has started (as opposed to being stuck in "loading"
}
```

**NB: In case multiple agents screen share with the same user, and there is an overlap, usage will be sent for both
agents separately.**

## Use in production
To use Upscope in production you will need a license key. This will ensure the server accepts more than a few connections
for development.

## Configuration options
The following options are available through env vars:

| Variable name | Description | Default |
| --- | --- | --- |
| `NODE_ENV` | The current environment | `development` |
| `SECRET_KEY` | A secret key to encrypt JWTs generated by the app. It should be secure and only known to the server. It can be changed at any time without repercussions. | `lazycat` (only configurable in production) |
| `LICENSE_KEY` | Your server license key. | `null` (required in production) |
| `REST_KEY` | The API key you will use to interact with the REST API | `upscope123` (only configurable in production) |
| `MONGO_URI` | Mongo connection URI | `mongodb://localhost:27017/upscope` |
| `REDIS_URI` | Redis connection URI | `redis://localhost:6379` |
| `REPORT_USAGE_ENDPOINT` | A url you use to keep track of your Upscope usage | `null` |
| `HIDE_IP_ADDRESS_FROM_AGENT` | If set to `yes` no IP address will be shown to agents | `no` |
| `HIDE_URL_FROM_AGENT` | If set to `yes` no visitor url will be shown to agents | `no` |
| `CUSTOM_AGENT_CSS` | Url to a CSS file to be included in the agent screen | `null` |
| `BYPASS_AUTH` | If set to `yes` agents will be redirected to the screen share without authentication | `no` |
| `PORT` | Port to run Upscope on | `5002` |
| `BASE_ENDPOINT` | Endpoint where Upscope server is running | `http://localhost:` + `PORT` |
| `HOMEPAGE` | Url to redirect agents to when they hit the `BASE_ENDPOINT` | `null` |
| `AUTH_ENDPOINT` | Url to redirect agents to when they follow the watch link for authentication | `null` |
| `THROTTLE_STATUS` | If set to `off` incoming connections will not be throttled | `on` |
| `DEBUG_COLLECTION_STATUS` | If set to `off` no debug info will be stored | `on` |
| `MAX_CONNECTIONS_PER_SECOND` | Maximum number of connections to accept per core per second | `50` |
| `MAX_DEBUG_PER_SECOND` | Maximum number of debug info to accept per core per second | `50` |
