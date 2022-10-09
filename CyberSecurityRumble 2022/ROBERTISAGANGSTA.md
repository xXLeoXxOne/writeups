# ROBERTISAGANGSTA

This challenge involved a simple login system. We also had the sources.

After I while I found out some things:

- You need an activation code to log in
- To be an admin you need an userid larger than 90mil.
- You can only execute the "date" command

## The Activation code

To sign up an account for the website, you need to enter an activation code into the form. 

```py
def check_activation_code(activation_code):
    # no bruteforce
    time.sleep(20)
    if "{:0>4}".format(random.randint(0, 10000)) in activation_code:
        return True
    else:
        return False
```

I quickly saw that we are not supposed to know the activation code, as it is random each time... but then I noticed something different: There is no length limit for the code & in the code it only checks if the random number is inside of the code!

That helped me to solve the first problem by generating a 5000+ digit long number to successfully sign up.

## Getting admin

The userid is created by concatting "1", the groupid and the userid from the form, which are restricted to 3 and 4 characters. Setting these numbers to 999 gave me the userid 19,999,999 which is still not high enough. I noticed that json.loads is used to put the userid into a json object

```py
userid = json.loads("1" + groupid + userid)
```

So I tried different things until I found out, that I can create a very high number by using the scientific notation! Setting my userid to `9e10` appends 10 zeros to the number which is more than enough. After signing up I am now admin and can access the admin page!

## Reading Files

The admin page contains a text field to put commands in and can print out the results after running it on the server.

```py
def validate_command(string):
    return len(string) == 4 and string.index("date") == 0
```

After a while I found out that there is actually no check if the string is actually a list of strings and the server expects the date command as a json object so after sending a list like `['date','','','']` I was still able to get the date command to run.

The manual of date gave me the final information, that I can actually read dates from files so I put a command together and got the flag by sending: `['date','-f','flag.txt','+%s']`!