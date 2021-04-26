Now we are basically done. Everything is working now we just need to deploy it.

The good thing is that the entire configuration is already done for how it should be deployed - in our `serverless.yml`.

The only thing serverless needs to deploy is some AWS credentials. The easiest way is to just type:

<pre>
$ serverless login
</pre>

which would normally open a web browser to log in to [serverless.com](https://serverless.com) and configure everything there. Unfortunately, at time of writing, opening a web browser seems to be beyond the current capabilities of Katacoda. 

But it is also possible to deploy with access key and secret key. I created an IAM User with Administrator Access and AWSCloudFormationFullAccess permission which is likely way more access than is necessary so proceed at your own risk and feel free to delete the user when you are done.

What you need to do now is type these commands but replace the environment variables with your own keys.
<pre>
export AWS_ACCESS_KEY_ID=your-key-here
export AWS_SECRET_ACCESS_KEY=your-secret-key-here

serverless deploy
</pre>

If everything worked correctly you should be able to see what endpoints you can reach your functions at. In my case it was:

<pre>
endpoints:
  GET - https://rpnbsqzo97.execute-api.us-east-1.amazonaws.com/dev/api/hello
  GET - https://rpnbsqzo97.execute-api.us-east-1.amazonaws.com/dev/api/secret
</pre>

feel free to test against those.