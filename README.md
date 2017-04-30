# JotunServerAuthorization
Authorization plugin for Kitura. Use tokens generated by username and password.

![Mac OS X](https://img.shields.io/badge/os-Mac%20OS%20X-green.svg?style=flat)
![Linux](https://img.shields.io/badge/os-linux-green.svg?style=flat)
![Apache 2](https://img.shields.io/badge/license-Apache2-blue.svg?style=flat)

## Summary
JotunServerAuthorization is an authentication middleware for Kitura. It makes posible to use benefits of auth token usage. But it is much simpler in compare to OAuth. 

## Example

To start using you just need to add `TokenAuthMiddleware` as a middleware. To initialize middle ware you have to create `JotunUsersStoreProvider` with your `JotunUserPersisting` implementation. But you can find in-memory store implementation so simple wrok example will could be like this:

```swift
let usersProvider = JotunUsersStoreProvider(persistor: JotunUserInMemoryPersistor.shared)
router.all("api/your_end_point", middleware: TokenAuthMiddleware(userStore: usersProvider))
```
After this Authorization header will be parsed and in case of errors `TokenAuthMiddleware` will make a proper response.
But where can you get that token? Framework propose you some code boilplate for that.

```swift
let usersProvider = JotunUsersStoreProvider(persistor: JotunUserInMemoryPersistor.shared)
let containerType = AuthHandlersCreator.ParametersContainer.querry
let handlersCreator = AuthHandlersCreator(userStore: usersProvider, parametersContainer: containerType)

router.get(basePath + "/login", handler: handlersCreator.loginHandler())
router.get(basePath + "/login/", handler: handlersCreator.loginHandler())

router.post(basePath + "/register", handler: handlersCreator.registerHandler())
router.post(basePath + "/register/", handler: handlersCreator.registerHandler())
```
With `containerType` you can customize where do you plan to put your parameters. 
Under the hood this routs makes redirets to the `JotunUsersStoreProvider` instanse. And that guy decide what to do and then just store information into passed `JotunUserPersisting` implementation. In example above it is `JotunUserInMemoryPersistor`.
You can check [CouchDB persistor](https://github.com/JotunHuset/JotunServer-Authorization-CouchDB). 

Cool, right? But I'm sure you have a question: "Where should I pass the username and password?". By default it is 
```
"jotun_userId" //key for username parameter
"jotun_userPassword" // key for user password parameter
```
And you can remap this parameters according to your API. Here is changes you could do
```swift
let parameters = AuthHandlersCreator.ParametersKeys(userId: "username", password: "password")
let handlersCreator = AuthHandlersCreator(userStore: usersProvider, parametersContainer: containerType, 
                                     parametersKeys: parameters)
```
What next? For sure we want to get a userId from passed token. `TokenAuthMiddleware` extends `Kitura.RouterRequest` to add `jotunUser` parameter. This means that any router called after middlware could access fulfilled `jotunUser`. 

# License

This library is licensed under Apache 2.0. Full license text is available in [LICENSE](LICENSE).
