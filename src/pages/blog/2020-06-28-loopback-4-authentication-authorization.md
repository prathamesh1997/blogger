---
templateKey: blog-post
title: Loopback 4 Authentication & Authorization
date: 2020-06-28T01:45:23.960Z
description: >-
  This article describes the details of loopback 4 Authentication and
  Authorization.

  The database used Firebase Authentication service.

  Below are the steps to configured authorization in Loopback 4
featuredpost: true
featuredimage: /img/loopback-sm.png
tags:
  - Loopback4
  - Backend
  - Typescript
---
* **In the first place** I create keys.ts file in the root of the project where I will write Token service bindings and Token service constants.

  `import{TokenService}from'@loopback/authentication';`

  `import{BindingKey}from'@loopback/context';`

  `import{FileUploadHandler}from'./types';`

  `exportnamespaceTokenServiceConstants{`

  `exportconstTOKEN_SECRET_VALUE='xxxxxxxxxxxxxxx';`

  `exportconstTOKEN_EXPIRES_IN_VALUE='600';`

  `}`

  `export namespace TokenServiceBindings{`

  `exportconstTOKEN_SECRET=BindingKey.create<string>(`

  `'authentication.jwt.secret',`

  `);`

  `exportconstTOKEN_EXPIRES_IN=BindingKey.create<string>(`

  `'authentication.jwt.expires.in.seconds',`

  `);`

  `exportconstTOKEN_SERVICE=BindingKey.create<TokenService>(`

  `'services.authentication.jwt.tokenservice',`

  `);`

  `}`
* **In the second place** I create authentication strategy src/authentication-strategy/authentication-strategy.ts, Authentication strategy will validate the Bearer token is in xxx.yyy.zzz format if not will throw an error.

  `import {asAuthStrategy, AuthenticationStrategy, TokenService} from '@loopback/authentication';`

  `import{bind,inject}from'@loopback/context';`

  `import{asSpecEnhancer,mergeSecuritySchemeToSpec,OASEnhancer,OpenApiSpec}from'@loopback/openapi-v3';`

  `import{HttpErrors,Request}from'@loopback/rest';`

  `import{UserProfile}from'@loopback/security';`

  `import{TokenServiceBindings}from'../keys';`

  `@bind(asAuthStrategy,asSpecEnhancer)`

  `exportclassJWTAuthenticationStrategyimplementsAuthenticationStrategy,OASEnhancer{`

  `name='jwt';`

  `constructor(@inject(TokenServiceBindings.TOKEN_SERVICE)publictokenService:TokenService) {}`

  `asyncauthenticate(request:Request):Promise<UserProfile|undefined> {`

  `consttoken:string=this.extractCredentials(request);`

  `constuserProfile:UserProfile=awaitthis.tokenService.verifyToken(token);`

  `returnuserProfile;`

  `}`

  `extractCredentials(request:Request):string{`

  `if(!request.headers.authorization) {`

  ``thrownewHttpErrors.Unauthorized(`Authorization header not found.`);``

  `}`

  `// for example : Bearer xxx.yyy.zzz`

  `constauthHeaderValue=request.headers.authorization;`

  `if(!authHeaderValue.startsWith('Bearer')) {`

  ``thrownewHttpErrors.Unauthorized(`Authorization header is not of type 'Bearer'.`);``

  `}`

  ``//split the string into 2 parts : 'Bearer ' and the `xxx.yyy.zzz` ``

  `constparts=authHeaderValue.split(' ');`

  `if(parts.length!==2)`

  `thrownewHttpErrors.Unauthorized(`

  `` `Authorization header value has too many parts. It must follow the pattern: 'Bearer xx.yy.zz' where xx.yy.zz is a valid JWT token.` ``

  `);`

  `consttoken=parts[1];`

  `returntoken;`

  `}`

  `modifySpec(spec:OpenApiSpec):OpenApiSpec{`

  `returnmergeSecuritySchemeToSpec(spec,this.name, {`

  `type:'http',`

  `scheme:'bearer',`

  `bearerFormat:'JWT'`

  `});`

  `}`

  `}`
* **In Third place** I create src/services/jwt-service.ts which includes verify and generate JSON web token.

  `import{TokenService}from'@loopback/authentication';`

  `import{inject}from'@loopback/context';`

  `import{HttpErrors}from'@loopback/rest';`

  `import{securityId,UserProfile}from'@loopback/security';`

  `import{promisify}from'util';`

  `import{TokenServiceBindings}from'../keys';`

  `import{FirebaseAdmin}from'./firebase-service';`

  ``

  `constjwt=require('jsonwebtoken');`

  `constsignAsync=promisify(jwt.sign);`

  `// const verifyAsync = promisify(jwt.verify);`

  `constadmin=FirebaseAdmin();`

  ``

  `exportclassJWTServiceimplementsTokenService{`

  `constructor(`

  `@inject(TokenServiceBindings.TOKEN_SECRET)privatejwtSecret:string,`

  `@inject(TokenServiceBindings.TOKEN_EXPIRES_IN)privatejwtExpiresIn:string`

  `) {}`

  ``

  `asyncverifyToken(token:string):Promise<UserProfile> {`

  `if(!token) {`

  ``thrownewHttpErrors.Unauthorized(`Error verifying token : 'token' is null`);``

  `}`

  ``

  `let userProfile:UserProfile;`

  ``

  `try{`

  `// decode user profile from token`

  `constdecodedToken=awaitadmin.auth.verifyIdToken(token);`

  `// don't copy over token field 'iat' and 'exp', nor 'email' to user profile`

  `userProfile=Object.assign(`

  `{[securityId]:'',name:''},`

  `{`

  `[securityId]:decodedToken.uid,`

  `name:decodedToken.email,`

  `id:decodedToken.uid`

  `}`

  `);`

  `}catch(error) {`

  ``thrownewHttpErrors.Unauthorized(`Error verifying token :${error.message}`);``

  `}`

  `returnuserProfile;`

  `}`

  ``

  `asyncgenerateToken(userProfile:UserProfile):Promise<string> {`

  `if(!userProfile) {`

  `thrownewHttpErrors.Unauthorized('Error generating token : userProfile is null');`

  `}`

  `constuserInfoForToken= {`

  `id:userProfile[securityId],`

  `name:userProfile.name,`

  `roles:userProfile.roles`

  `};`

  `// Generate a JSON Web Token`

  `lettoken:string;`

  `try{`

  `token=awaitsignAsync(userInfoForToken,this.jwtSecret, {`

  `expiresIn:Number(this.jwtExpiresIn)`

  `});`

  `}catch(error) {`

  ``thrownewHttpErrors.Unauthorized(`Error encoding token :${error}`);``

  `}`

  ``

  `returntoken;`

  `}`

  `}`
* **In Fourth place** register authentication straategy in application.ts file and add auth component.

  `import{JWTAuthenticationStrategy}from'./authentication-strategy/jwt-strategy';`

  `import{`

  `TokenServiceBindings,`

  `TokenServiceConstants,`

  `}from'./keys';`

  `import{JWTService}from'./services/jwt-service';`

  Adding inside the constructor of application.ts file

  `// adding auth component`

  `this.component(AuthenticationComponent);`

  ``

  `// register authentication strategy`

  ``

  `this.add(createBindingFromClass(JWTAuthenticationStrategy));`

  `this.bind(TokenServiceBindings.TOKEN_SECRET).to(`

  `TokenServiceConstants.TOKEN_SECRET_VALUE,`

  `);`

  `this.bind(TokenServiceBindings.TOKEN_EXPIRES_IN).to(`

  `TokenServiceConstants.TOKEN_EXPIRES_IN_VALUE,`

  `);`

  `this.bind(TokenServiceBindings.TOKEN_SERVICE).toClass(JWTService);`
* **In Fifth place** create handle function in sequence.ts file 

  `import{`

  `AuthenticateFn,`

  `AuthenticationBindings,`

  `AUTHENTICATION_STRATEGY_NOT_FOUND,`

  `USER_PROFILE_NOT_FOUND,`

  `}from'@loopback/authentication';`

  `import{inject}from'@loopback/context';`

  `import{`

  `FindRoute,`

  `InvokeMethod,`

  `InvokeMiddleware,`

  `ParseParams,`

  `Reject,`

  `RequestContext,`

  `RestBindings,`

  `Send,`

  `SequenceHandler,`

  `}from'@loopback/rest';`

  `asynchandle(context:RequestContext) {`

  `try{`

  `const{request,response} =context;`

  `constfinished=awaitthis.invokeMiddleware(context);`

  `if(finished)return;`

  `constroute=this.findRoute(request);`

  `awaitthis.authenticateRequest(request);`

  `constargs=awaitthis.parseParams(request,route);`

  `constresult=awaitthis.invoke(route,args);`

  `this.send(response,result);`

  `}catch(err) {`

  `if(`

  `err.code===AUTHENTICATION_STRATEGY_NOT_FOUND||`

  `err.code===USER_PROFILE_NOT_FOUND`

  `) {`

  `Object.assign(err, {statusCode:401/* Unauthorized */});`

  `}`

  `this.reject(context,err);`

  `}`

  `}`
* **Now How to use authorization on Rest end points? Its very simple lets follow this steps.**

  create new file inside utils/security-specs.ts add this code 

  `exportconstOPERATION_SECURITY_SPEC= [ {jwt:[] } ];`

  Next is to use <!--StartFragment-->

  Syntax:`@authenticate(strategyName: string, options?: object)`

  Marks a controller method as needing an authenticated user. This decorator requires a strategy name as a parameter.

  `@get('/ping', {`

  `security:OPERATION_SECURITY_SPEC,`

  `responses:{`

  `'200':PING_RESPONSE`

  `}`

  `})`

  `@authenticate('jwt')`

  `asyncprintCurrentUser(`

  `@inject(SecurityBindings.USER, {optional:true})`

  `currentUserProfile:UserProfile`

  `):Promise<Profile> {`

  `currentUserProfile.uid=currentUserProfile[securityId];`

  `returncurrentUserProfile.uid;`

  `}`

  This is all, following this steps you must have authentication and authorization working very well. <!--StartFragment-->

  Good luck!