# Overview
>Business logic vulnerabilities are flaws in the design and implementation of an application that allow an attacker to elicit unintended behavior. This potentially enables attackers to manipulate legitimate functionality to achieve a malicious goal. These flaws are generally the result of failing to anticipate unusual application states that may occur and, consequently, failing to handle them safely.

## Examples
- Check POST and GET parameters, if a sensitive parameter is used : try to modify its value
	- For instance, if there is an integer parameter, change its value to `-1`, `0`, `1`, etc.
	- For coupon codes : alternate between 2 (or more) codes to bypass the control.