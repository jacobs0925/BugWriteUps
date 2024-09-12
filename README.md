# Bug Writeups

Here's a little summary on the first bug report, I hope to be adding more to this page soon!

## Permission Escalation via CSRF
Non-internal members can sign in at the internal portal, gaining unauthorized access to the system.


Certain sensitive pages enumerated by the _buildManifest.js file were able to be accessed with default permissions. In one of these pages, an employee at each franchise would upload images of products for an audit and this page returned a response containging each of these employees' `userId` and `storeId`.

When determining what administrator apps are loaded for the user, a request is made to the backend with the user's `userId` as a variable. An attacker can inject a corporate user's `userId` into this query, retrieving their permissions and escalating privileges. This would result in a permisisons JSON file. In the file there are objects such as the following:

```json
{
    "type": "GROUP",
    "tool": "FILE_UPLOAD",
    "__typename": "InternalToolSource"
}
```

In the _app.js files, all of the tools are enumerated so an attacker can further escalate their permissions by simply injecting it into the response of the permission query request. Many of the truly compromising pages like the emplyee directory or the discount creation page were unable to be accessed wkith these tools and likely required a permission in the form:

```json
{
    "node": {
        "grantedPermissionId": "XXX:PermissionsGranted",
        "permissionId": "XXX:Permissions",
        "groupId": "XXX:PermissionGroups",
        "permission": "OPS_MANUAL:READ",
        "scopeId": null,
        "scope": "GLOBAL",
        "type": "GROUP",
        "__typename": "GrantedPermission"
    },
    "__typename": "GrantedPermissionEdge"
}
```

I could not find these enumerated in the JS or brute force them but if an attacker were to be able to reverse engineer them, they would have total access to all corporate administrator features.

Various internal tools are exposed with the permisisons I was able to obtain.The would allow an attacker to upload files, delete customer suggestions, and access sensitive customer information, such as phone numbers.
- The **File Upload** page allows anyone to upload files and returns URLs to these files, making it possible to distribute malicious files through the platform. Additionally, it provides a shortened link in the form \<COMPANY\>.co/1345. Naturally this would make a convincing link to send employees and run malicious JS on their machines.
- The **Suggestions** page exposes customer phone numbers and allows suggestions to be deleted without authorization.
- The **Store** page leaks information about all store locations in a query
- The **Phone Logs** page leaks employee phone numbers.
