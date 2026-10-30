# Integrity Checks

|ID          |
|------------|
|WSTG-BUSL-03|

## Summary

Many applications display different fields depending on the user or situation by hiding some inputs, or present values as non-editable controls or fixed drop-down lists. These controls are only enforced in the browser, and it is possible to submit different values to the server using a proxy. Server-side controls must perform relational and authorization checks to ensure that the data accepted is valid for the user and the business process. If the application exposes values related to business rules, such as quantity, as non-editable fields, it must maintain a copy on the server side and use that copy for business logic processing.

Business logic integrity check vulnerabilities are unique in that these misuse cases are application specific: users should only be able to write, update or edit specific artifacts at specific times, as the business process logic dictates. System artifacts such as logs must also be protected from unauthorized read, write and removal.

## Test Objectives

- Review the project documentation for components of the system that move, store, or handle data.
- Determine what type of data is logically acceptable by the component and what types the system should guard against.
- Determine who should be allowed to modify or read that data in each component.
- Attempt to insert, update, or delete data values used by each component that should not be allowed per the business logic workflow.

## How to Test

### Hidden Fields

- Using a proxy capture HTTP traffic looking for hidden fields.
- If a hidden field is found, see how these fields compare with the GUI application and start interrogating this value through the proxy by submitting different data values trying to circumvent the business process and manipulate values you were not intended to have access to.
- Check whether values that the client holds as state, such as signed cookies or tokens, are verified by the server and whether altering them is detected (see [Testing JSON Web Tokens](../06-Session_Management/10-JSON_Web_Tokens.md)).

### Non-Editable Controls

- Using a proxy capture HTTP traffic looking for a place to insert information into areas of the application that are non-editable.
- If it is found, see how these fields compare with the GUI application and start interrogating this value through the proxy by submitting different data values trying to circumvent the business process and manipulate values you were not intended to have access to.

### Logs and Data Stores

- List components of the application or system that could be impacted, for example logs or databases.
- For each component identified, try to read, edit or remove its information. For example log files should be identified and testers should try to manipulate the data/information being collected, including by submitting values that contain line breaks or other control characters.

## Examples

### Role-Restricted Form Fields

Imagine an application that only allows the admin user to change the password for other users in the system. The admin user will see the username and password fields to enter a username and password while other users will not see either field. However, if a non admin user submits information in the username and password field through a proxy they may be able to "trick" the server into believing that the request has come from an admin user and change the password of other users.

### Unauthorized Drop-Down Value

Most web applications have dropdown lists making it easy for the user to quickly select their state, month of birth, etc. Suppose a Project Management application allowed users to login and depending on their privileges presented them with a drop down list of projects they have access to. What happens if an attacker finds the name of another project that they should not have access to and submits the information via a proxy. Will the application give access to the project? They should not have access even though they skipped an authorization business logic check.

### Unverified Self-Service Updates

Suppose the motor vehicle administration system required an employee initially verify each citizen's documentation and information when they issue an identification or driver's license. At this point the business process has created data with a high level of integrity as the integrity of submitted data is checked by the employees. Now suppose the application is moved to the internet so employees can log on for full service or citizens can log on for a reduced self-service application to update certain information. At this point an attacker may be able to use an intercepting proxy to add or update data that they should not have access to and they could destroy the integrity of the data by stating that the citizen was not married but supplying data for a spouse's name. This type of inserting or updating of unverified data destroys the data integrity and might have been prevented if the business process logic was followed.

### Log Injection

Many systems include logging for auditing and troubleshooting purposes. If the application writes a user-supplied value, such as a username, to the log without neutralizing line breaks, an attacker can submit a value containing a line break followed by text that looks like a separate log entry. The forged entry can mislead anyone reviewing the log. See [CWE-117: Improper Output Neutralization for Logs](https://cwe.mitre.org/data/definitions/117.html).

## Related Test Cases

- All [Injection Testing](../07-Injection/README.md) test cases.
- [Business Logic Data Validation](01-Business_Logic_Data_Validation.md)
- [Ability to Forge Requests](02-Ability_to_Forge_Requests.md)
- [Bypassing Authorization Schema](../05-Authorization/02-Bypassing_Authorization_Schema.md)
- [Privilege Escalation](../05-Authorization/03-Privilege_Escalation.md)
- [Insecure Direct Object References](../05-Authorization/04-Insecure_Direct_Object_References.md)

## Remediation

The application should follow strict access controls on how data and artifacts can be modified and read, and through trusted channels that ensure the integrity of the data. Proper logging should be set in place to review and ensure that no unauthorized access or modification is happening.

## Tools

- [ZAP](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)
- [Caido](https://caido.io)

## References

- [CWE-345: Insufficient Verification of Data Authenticity](https://cwe.mitre.org/data/definitions/345.html)
- [CWE-117: Improper Output Neutralization for Logs](https://cwe.mitre.org/data/definitions/117.html)
- [OWASP Cheat Sheet Series - Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [OWASP Application Security Verification Standard (ASVS) 5.0](https://github.com/OWASP/ASVS/tree/v5.0.0)
