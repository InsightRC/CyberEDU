# monty-mole

### Category: Web | Difficulty: Easy | Tenant: ROCSC

The description hints to the fact that we can't bruteforce or scan repeatedly.

What we have to do here is test the parameters of the form using sqlmap, but not by giving it the url. The POST request can be given using the -r option.

To do that, first you have to intercept the request using Burp, copy-paste it to a file and then run the command. (don't forget to add the --dump-all option too, or else you won't get anything)
