# 2.1 Application Overview
> Importance: Low

there is a table listing 3 components, the third line has a link to [Data](https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api/raw/master/data.tar.gz) which is broken, but is not required as well.

---
# 2.5 Deploy Ratings API
> Importance: High

## changes in app.js

### Remove connectOptions from 
``` js
// var promise = mongoose.connect(process.env.MONGODB_URI, connectOptions);
var promise = mongoose.connect(process.env.MONGODB_URI)
```

### check for undefined, not only 0
``` js
// find (in two places)
// if (c == 0)  and replace it with below
if (c === undefined || c == 0) {

```
---
## changes in package.json
change `"mongoose": "^4.13.8" ` to `"mongoose": "^6.1.8" `

--- 

# 2.6 Deploy Ratings frontend
> Importance: High

## package.json
upgrade ` "node-sass": "^4.12.0", ` to  ` "node-sass": "^4.14.0", `

## Dockerfile change base image
`FROM node:13.5-alpine` to 
`FROM node:14-alpine`

## Dockerfile change python
change 
`RUN apk update && apk add python g++ make && rm -rf /var/cache/apk/*`
to 
`RUN apk update && apk add python3 g++ make && rm -rf /var/cache/apk/*`
