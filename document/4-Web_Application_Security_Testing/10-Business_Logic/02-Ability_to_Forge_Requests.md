# Ability to Forge Requests

|ID          |
|------------|
|WSTG-BUSL-02|

## Summary

Forging requests is a method that attackers use to circumvent the frontend GUI application to directly submit information for backend processing. The goal of the attacker is to send HTTP POST/GET requests through an intercepting proxy with data values that are not supported, guarded against, or expected by the application's business logic. Some examples of forged requests include exploiting guessable or predictable parameters or exposing "hidden" features and functionality such as enabling debugging or presenting special screens or windows that are very useful during development but may leak information or bypass the business logic.

Vulnerabilities related to the ability to forge requests are unique to each application and different from business logic data validation in that its focus is on breaking the business logic workflow.

Applications should have logic checks in place to prevent the system from accepting forged requests that may allow attackers the opportunity to exploit the business logic, process, or flow of the application. Request forgery is nothing new; the attacker uses an intercepting proxy to send HTTP POST/GET requests to the application. Through request forgeries attackers may be able to circumvent the business logic or process by finding, predicting and manipulating parameters to make the application think a process or task has or has not taken place.

Also, forged requests may allow subversion of programmatic or business logic flow by invoking "hidden" features or functionality, such as debug options and feature flags initially used by developers and testers, or undocumented admin or test functions (sometimes referred to as an ["Easter egg"](https://en.wikipedia.org/wiki/Easter_egg_(media))).

## Test Objectives

- Review the project documentation looking for guessable, predictable, or hidden functionality of fields.
- Insert logically valid data in order to bypass normal business logic workflow.

## How to Test

### Through Identifying Guessable Values

- Using an intercepting proxy observe the HTTP POST/GET looking for some indication that values are incrementing at a regular interval or are easily guessable.
- If it is found that some value is guessable this value may be changed and one may gain unexpected visibility.

### Through Identifying Hidden Options

- Using an intercepting proxy observe the HTTP POST/GET looking for some indication of hidden features such as debug that can be switched on or activated.
- If any are found try to guess and change these values to get a different application response or behavior.
- Look for parameters the interface never sends, such as `debug`, `test`, `admin` or `role`, by adding them to requests and comparing responses. Parameter discovery tools can automate the guessing.
- See also [Mass Assignment](../07-Injection/20-Mass_Assignment.md) and [HTTP Parameter Pollution](../07-Injection/04-HTTP_Parameter_Pollution.md).

### Through Replaying Requests Out of Order

- Replay captured requests in a different order, or skip steps, to see whether the server relies on the client to enforce the sequence (see [Circumvention of Work Flows](06-Circumvention_of_Work_Flows.md)).

## Examples

### Reusing a One-Time Discount

Suppose an e-commerce theater site allows users to select their ticket, apply a onetime 10% Senior discount on the entire sale, view the subtotal and tender the sale. If an attacker is able to see through a proxy that the application has a hidden field (of 1 or 0) used by the business logic to determine if a discount has been taken already or not. The attacker is then able to submit the 1 or "no discount has been taken" value multiple times to take advantage of the same discount multiple times.

### Development Flags in an Online Game

Suppose an online video game pays out tokens for points scored for finding pirate's treasure, pirates, and for each level completed. These tokens can later be exchanged for prizes. Additionally each level's points have a multiplier value equal to the level. If an attacker was able to see through a proxy that the application has a hidden field used during development and testing to quickly get to the highest levels of the game they could quickly get to the highest levels and accumulate unearned points quickly.

Also, if an attacker was able to see through a proxy that the application has a hidden field used during development and testing to enable a log that indicated where other online players, or hidden treasures were in relation to the attacker, they would then be able to quickly go to these locations and score points.

## Related Test Cases

- [Business Logic Data Validation](01-Business_Logic_Data_Validation.md)
- [Integrity Checks](03-Integrity_Checks.md)
- [Circumvention of Work Flows](06-Circumvention_of_Work_Flows.md)
- [Mass Assignment](../07-Injection/20-Mass_Assignment.md)
- [Exposed Session Variables](../06-Session_Management/04-Exposed_Session_Variables.md)
- [Cross Site Request Forgery (CSRF)](../06-Session_Management/05-Cross_Site_Request_Forgery.md)
- [Account Enumeration and Guessable User Account](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md)

## Remediation

The application must be smart enough and designed with business logic that will prevent attackers from predicting and manipulating parameters to subvert programmatic or business logic flow, or exploiting hidden/undocumented functionality such as debugging.

## Tools

- [ZAP](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)
- [Caido](https://caido.io)
- [Arjun](https://github.com/s0md3v/Arjun) (HTTP parameter discovery)
- [Param Miner](https://github.com/PortSwigger/param-miner) (Burp extension for hidden parameter discovery)

## References

- [CWE-840: Business Logic Errors](https://cwe.mitre.org/data/definitions/840.html)
- [OWASP Application Security Verification Standard (ASVS) 5.0](https://github.com/OWASP/ASVS/tree/v5.0.0)
- [OWASP Cheat Sheet Series - Business Logic Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)
