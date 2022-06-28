# pnscan-frontend
The web frontend proejct for [pnscan-cloud][]. you can check main purpose of pnscan-cloud and pnscan-frontend on pnscan-cloud repo.

## Getting started
### 0. Setup first backend project
This project is a frontend project. You need to set up the backend first.

https://github.com/planetarium/pnscan-cloud

Node v16.5

### 1. Pull this project to local

```
npm install
```

### 2. Create and fill .env file

You need to create the files below.
```
.config
.env.localhost
.env.production
```

.config
```
{
    # Required. AWS Region
    "region": "",
    # Required. AWS Credentials to upload s3 bucket
    "credentials": {
        "accessKeyId": "",
        "secretAccessKey": ""
    },
    "s3WebBucketName": ""
}
```

.env.localhost , .env.production
```
VUE_APP_MODE= [DEVEL | PRODUCTION]
VUE_APP_API_HOST=
VUE_APP_WS_ENDPOINT=
```
The above properties can be found in the `9cscan-cloud/.deploy` file.


### 3. Run local
```
grunt serve
```

### 4. Deploy
```
grunt deploy 
```


## License

Apache 2.0
