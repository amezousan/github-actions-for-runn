# github-actions-for-runn
This is an example of github actions for [Runn](https://github.com/k1LoW/runn)

## How to test it around?
You need to clone this repo at first ;)

### Run your github actions locally
[Act](https://github.com/nektos/act) is one of my favorite tools to run Github Actions locally. The installation is pretty simple when you use Mac with brew: `brew install act`

And you can create `.secrets` file for Act ( The value below is made of `echo -n "postman:password"|base64` ):

```sh:.secret
POSTMAN_BASIC_AUTH=cG9zdG1hbjpwYXNzd29yZA==
```

After you've launched Docker Desktop, then run the command: `act`:

```sh
$ pwd
<your-path-to>/github-actions-for-runn
$ tree -a -L 1
.
├── .git
├── .github
├── .gitignore
├── .secrets
├── LICENSE
├── README.md
└── test-scenarios
$ act
...
| 1 scenario, 0 skipped, 0 failures
[Backend API Tests/run-test]   ✅  Success - Main Run tests with Runn
[Backend API Tests/run-test] 🏁  Job succeeded
```

Now you should see the result of the workflow :) Welcome to a DevOps world :D

### Run on Github Actions
* Create a new secret, `POSTMAN_BASIC_AUTH` with the value `cG9zdG1hbjpwYXNzd29yZA==`

* The workflow by default will be triggered whenever you push your commit into the repo. However you can schedule it to trigger with cron ([Ref](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)):
    ```
    on:
        schedule:
            # * is a special character in YAML so you have to quote this string
            - cron:  '30 5,17 * * *'
    ```

### Use Runn locally
You can also use [Runn](https://github.com/k1LoW/runn) to test your scenarios: e.g., `runn run --debug test-scenarios/*`

However, there are a few tips to use it properly:

#### Define a default value in variables
In some cases, I've seen such an error when I loaded an undefined-variable in my scenarios:

```sh
Run 'req' on 'Http Request Examples on Postman'.steps.read
panic: interface conversion: interface {} is nil, not string

goroutine 1 [running]:
github.com/k1LoW/runn.parseHTTPRequest(0x49?)
```

This is because Runn sometimes cannot handle a null value, so you can avoid such an error by defining default values.

```yml
diff --git a/test-scenarios/crudPostman.yml b/test-scenarios/crudPostman.yml
index 25240f6..8d4b28d 100644
--- a/test-scenarios/crudPostman.yml
+++ b/test-scenarios/crudPostman.yml
@@ -4,7 +4,7 @@ runners:
 vars:
   foo1: bar1
   foo2: bar2
-  base64Auth: "${POSTMAN_BASIC_AUTH}" # echo -n "postman:password"|base64
+  base64Auth: "${POSTMAN_BASIC_AUTH:-none}" # echo -n "postman:password"|base64
```

#### Pass a value for the variable
You can do either:
* `export POSTMAN_BASIC_AUTH=cG9zdG1hbjpwYXNzd29yZA==` on your shell OR
* Put that one into your shell profile: e.g., `~/.zshrc`