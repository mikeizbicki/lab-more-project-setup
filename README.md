# More project setup

<img width=400px src=img/boromir.png />

This lab will cover more "behind the scenes" details for getting python projects setup.
You will learn:
1. how to write "test cases for your test cases"
1. how to get LLMs working in github actions
1. how to let other people "pip install" your projects

The code you write for this lab will be the basis of your next project.

## Tasks

This lab continues from the `llm-inclass` project we were building in class.
I encourage you to use the version of the code that you have been writing.
But if that code does not work for whatever reason,
you are welcome to clone my version of the repo <https://github.com/mikeizbicki/llm-inclass>.

### Code Coverage

So far in class we've been doctests to ensure that our code is correct.
And I've generally given you a good set of doctests for all the functions you had to write.
But soon, you are going to have to write your own doctests.

**Q:** How can you ensure that our doctests are correct?

**A:** Writing test cases for your test cases.

<img width=400px src=img/dawg.png />

*Code coverage* is a simple tool for "testing our test cases".
The idea is:
1. run the doctests
1. count the number of lines that python "looked at" when doing the doctests
1. any lines that python didn't look at are lines that haven't been tested
1. write more doctests that test those lines

This will make more sense with some examples.
To start, we'll need to install some libraries:
```
$ pip install pytest coverage supply-chain-attack-poc
```
What they do:
1. `pytest` is a standard library for testing in python.
    There are dozens of different styles of test cases (beyond just doctests),
    and all of these advanced testing techniques are built on this library.
1. `coverage` is the extension to pytest that we'll be working with.
    This is the particular library that will tell us if our test cases are "good enough".
1. `supply-chain-attack-poc` is a library written by me.
    First some background.
    <https://pypi.org> is the *package repository* that stores all of the python libraries that can be "pip installed".
    It is important to remember that anyone, anywhere on earth can upload anything they want to pypi.

    > **NOTE:**
    > PyPI stands for the **Py**thon **P**ackage **I**ndex.
    > It is officially pronounced like "pie - pee - eye".

    At the end of this lab, you will be uploading your own project to pypi.
    What the `supply-chain-attack-poc` repo does is it plays the rickroll video when you install it.
    This is just a friendly/fun reminder that when you install my library,
    you are giving me full access to your computer.
    I could have run absolutely anything.
    Malware is regularly distributed to programmers in this way.
    And companies are regularly hacked in this way.

With those libraries installed, you can now run the coverage tests:
```
$ coverage run -m pytest chat.py --doctest-modules
================================= test session starts =================================
platform linux -- Python 3.13.5, pytest-9.0.2, pluggy-1.6.0
rootdir: /home/user/proj/cmc-csci040/docsum
plugins: anyio-4.13.0
collected 1 item                                                                      

chat.py .                                                                       [100%]

================================== 1 passed in 1.26s ==================================
```
Note that the coverage tests also run the doctests,
and you need to make sure that your doctests are passing for the coverage tests to make any sense.

> **WARNING:**
> The coverage command above creates a file `.coverage`.
> This is a "bad" file that we do not want added to the repo.
> If this file ends up on github, they you will lose points on the lab.
> I encourage you to add it `.gitignore`.

We can view the results of our coverage tests by running the command
```
$ coverage report -m
Name      Stmts   Miss  Cover   Missing
---------------------------------------
chat.py      24      9    62%   58-66
---------------------------------------
TOTAL        24      9    62%
```
The important part of this output is the number `62%`.
(Your number will probably be slightly different if you are using your own version of the code.  That's okay.)
This means that 62% of the lines in your code were tested by the doctest.

To see which lines were tested, and which lines were not tested, run the following commands
```
$ coverage html
Wrote HTML report to htmlcov/index.html
$ firefox htmlcov/index.html
```

> **WARNING:**
> The commands above create more files that we do not want added to our repo.

After loading the `htmlcov/index.html` file, you can click on the `chat.py` link.
You will see a webpage that looks something like

<img src=img/coverage.png />

In the output above, the green lines are all lines that were run with the doctests.
You'll observer that all of the contents of the `Chat` class are in green because the doctests for the chat class make all of those lines run.
And since the doctests are passing, that means that everything about the `Chat` class is working and does what we want it to do.

The problem is the red lines at the bottom.
We don't have any doctests for this code:
```
    import readline
    chat = Chat()
    try:
        while True:
            user_input = input('chat> ')
            response = chat.send_message(user_input)
            print(response)
    except KeyboardInterrupt:
        print()
```
We "manually" tested it to prove that it does what we want in class,
but we have no way to "automatically" test it.

Automatic test cases are useful because:
1. they make working on large projects much easier
1. you can prove to me (Mike) that your homeworks are correct

    (I never actually run other people's code because I don't want to be rickrolled/hacked.)

So our goal is to make an automatic test case that proves that the REPL (Read Eval Print Loop) above works.

This is a 2 step process.

First, we refactor the code in the if statement into its own function:
```
def repl():
    import readline
    chat = Chat()
    try:
        while True:
            user_input = input('chat> ')
            response = chat.send_message(user_input)
            print(response)
    except KeyboardInterrupt:
        print()


if __name__ == '__main__':
    repl()
```
Then we write doctests for that new function.

But wait!

There's something about this function that makes writing doctests tricky.
Before reading on, try to write your own doctests for this function to figure out why it is hard to do.

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

The problem with the function above is that it takes *user input* with the `input` function.
All of the functions we've tested before take input via *parameters*,
and those are easy to supply in the function call in the doctest.
But `input` reads from the keyboard,
and there is no keyboard available when running doctests.

So how do we fix this?

<img width=400px src=img/monkey-patch.webp />

The answer is called *monkey patching*.
Monkey patching is a technique for temporarily editing the behavior of a python function outside of the function.
In this case, we are going to change the behavior of the python `input` function to make it compatible with doctests.

To see how it works, first open up interactive python and run the following code.
First observe that the input function works normally in the python interactive terminal:
```
>>> x = input('chat> ')
chat> hello world
>>> print(x)
hello world
```
Now we are going to define our own new function `monkey_input` and overwrite the original `input` function with this new `monkey_input` function.
```
>>> def monkey_input(prompt):
...     user_input = 'hola mundo'
...     print(f'{prompt}{user_input}')
...     return user_input
>>> input = monkey_input        # this is the monkey patch
>>> x = input('chat> ')
chat> hola mundo
>>> print(x)
hola mundo
```
After doing the monkey patch, whenever the code calls `input`, it goes to our new modified `monkey_input` function instead of the original built-in function.

That's how monkey patching works in principle.
There's two more subtleties we need to consider to use monkey patching to test our `repl` function:

1. If we monkey patch our `repl` function with the `monkey_input` function above,
    then we will enter an infinite loop because `input` will always return the same results and never raise the `KeyboardInterrupt`.
    Here is a better `monkey_input` function for testing.
    ```
    def monkey_input(prompt, user_inputs=['Hello, I am monkey.', 'Goodbye.']):
        try:
            user_input = user_inputs.pop(0)
            print(f'{prompt}{user_input}')
            return user_input
        except IndexError:
            raise KeyboardInterrupt
    ```
    Try copy/pasting this function into interactive python, then run
    ```
    >>> monkey_patch('chat> ')
    >>> monkey_patch('chat> ')
    >>> monkey_patch('chat> ')
    ```
    You should notice different results from each call to `monkey_patch`.
    It should also make sense why you get different results based on memory management issues,
    and it should also make sense why this would stop the infinite loop.

1. Next, a simple monkey patch of `input = monkey_input` won't actually modify the behavior of the repl function.

    The reason is another subtle memory management issue:
    The `input` variable inside the doctest is in a different *frame* as the `input` variable inside the `repl` function,
    and so reassigning the variable in one frame does not reassign it in another frame.
    Instead of just reassigning the variable, we must actually modify the object directly.
    (It's okay if the above doesn't make 100% perfect sense to you, but it is the technically correct language.)

    Python provides a module called `builtins` that allows for directly modifying the objects that functions point to.
    Inside the doctest, we monkey patch the `input` function like:
    ```
    >>> import builtins
    >>> builtins.input = monkey_input
    ```

Combining both of these two techniques together,
we get a docstring that looks something like:
```
'''
>>> def monkey_input(prompt, user_inputs=['Hello, I am monkey.', 'Goodbye.']):
...     try:
...         user_input = user_inputs.pop(0)
...         print(f'{prompt}{user_input}')
...         return user_input
...     except IndexError:
...         raise KeyboardInterrupt
>>> import builtins
>>> builtins.input = monkey_input
>>> repl()
'''
```
Note that you will have to run the doctests and observe the actual output for your llm code in order to find the correct output of `repl` for your doctest.
Don't forget that you will have to set the temperature somewhere to make this work.

Once you've got it working, rerun the coverage checks
```
$ coverage run -m pytest chat.py --doctest-modules
$ coverage report -m
Name      Stmts   Miss  Cover   Missing
---------------------------------------
chat.py      26      1    96%   79
---------------------------------------
TOTAL        26      1    96%
$ coverage html
Wrote HTML report to htmlcov/index.html
$ firefox htmlcov/index.html
```
You should observe that there is now only one line not covered by your doctests,
and a much higher coverage percentage!

As you can see, getting high coverage for your code can be difficult and time consuming.
Higher is always better, but for most real-world projects, getting 100% coverage is unrealistic.

<img width=45% src=img/no-time-for-that.jpg /><img width=45% src=img/jedi-tests.jpg />

### API keys with github actions

If you don't already have a github repo for your project.
Create one and push your code to github.

Recall that committing/pushing regularly is a habit that all good developers are in.

Now create a github action that runs your doctests.
And commit/push that to github.

You should observe that the action fails because the github action does not have access to your `GROQ_API_KEY` environment variable,
but that is needed for your tests.

<img width=400px src=img/env.jpeg />

Recall that we should never place API keys (or other secrets) in a git repo.
This presents a problem for our continuous integration setup,
because our doctests must connect to the groq API to work.

Github's solution to this problem is called *Secrets*.

For this section of the lab, you will follow github's tutorials on registering your API key with github as a secret so that the doctests can run successfully.

> **NOTE:**
> These tutorials are not written for this class,
> but generic tutorials for professional programmers.
> You may have to modify the tutorials just slightly to get them to work for your code.
> Part of the purpose of this lab is to not just learn about github secrets,
> but to "learn how to learn" by following real-world tutorials.
>
> Remember: real-world programmers have to learn new concepts everyday.
> If you can't learn new concepts on your own without being explicitly taught in a classroom environment,
> you will not succeed as a data scientist or software engineer.

There are two steps:

1. Follow these instructions to store your `GROQ_API_KEY` variable as a secret within your github repo: <https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets>

1. Follow these instructions to modify the github action to load the secrets into environment variables: <https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets#using-secrets-in-a-workflow>.

Once you have the doctests passing in github actions,
add the code coverage commands below to your github action:
```
$ coverage run -m pytest chat.py --doctest-modules
$ coverage report -m
```
This will allow me to view your code coverage by looking at the output of your actions.
Part of your grade for the assignment will be that I observer >90% code coverage in the action.

### Upload to PyPi

Recall that *anyone* can upload any project to pypi to make it installable with `pip`.
That includes you!

Follow [this tutorial](https://realpython.com/pypi-publish-python-package/) to upload your package.

> **NOTE:**
> The tutorial above is relatively long.
> You are welcome to skim to find only the relevant parts; you do not have to complete the whole thing step-by-step.
> The most important parts are:
> 1. You will also need to create a `pyproject.toml` file.
>    Inside this file you will need to specify the "script" that gets created, which should be your `chat.py` file.
>    Call the script `chat`, and this is the command you will be able to run after your library has been pip-installed to execute your program.
> 1. There is a command called `twine` that you will use to actually upload your library to pypi.

> **NOTE:**
> You will have to select a name for your project.
> Choose a name of the form like `cmc-csci005-yourname`.

> **WARNING:**
> The commands in the tutorial above will create many bad files that we do not want added to our repo!
>
> At this point, you have seen that there are many files that get created at many points in the development process that we do not want added to our git repo.
> Github maintains a list of useful `.gitignore` files for different types of projects.
> You might consider using their file, which you can find at <https://github.com/github/gitignore/blob/main/Python.gitignore>.

### Integration Tests

**Q:** How do we know that everything is uploaded to pypi correctly?

**A:** We test it!

The doctests are an example of what are called *unit tests*:
they test just one "unit" of your program at a time (typically a function/class).
An *integration test* is a test that runs everything together all at once.
It ensures these individual units work well with each other.
They typically run everything in the code "end-to-end",
and use the same interface that a user sees when running the program.

<img width=300px src=img/integration.webp />

The following code provides a simple example of an integration test for this project:
```
$ pip3 install <your-library-name>
$ chat <<'EOF'
I am bob.
What is my name?
EOF
```
This test ensures that your library can be successfully installed and that the `chat` command works to run it.
But notice that it doesn't actually check what the output is.
A more complicated integration test could verify that we get the "correct" output,
but this simple integration test just verifies that we don't get any errors,
which is good enough for us for now.

> **NOTE:**
> One reason why programmers like terminal programs is because they are easy to write integration tests for.
> Web applications and mobile apps, in contrast, are very hard to write integration tests for.
> For example, it is hard to programmatically move the mouse around to click various buttons the way a user might,
> and it is hard to verify that the program did not generate any errors after a button click has happened.

Create a new github action called `integration-tests` that runs the simple integration test above.

## Submission

Submit to canvas the url to your repo.

You must have a README file that
1. Contains a green `doctest` badge.
    (I should be able to look at the corresponding action and see >90% coverage.)
1. Contains a green `integration-test` badge.
1. A 1-2 sentence description of your project.
1. A link to your pypi project page.

Recall that you will lose points for any unnecessary files contained in your git repo.
