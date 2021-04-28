Now we can create a file where our Lambda functions will exist. Use the following command:

`touch handler.ts`{{execute}}

Let's type the following code in `handler.ts` or simply press the "Copy to Editor" button:

<pre class="file" data-filename="project-red/handler.ts" data-target="replace">
import { APIGatewayProxyResult } from 'aws-lambda';

export const hello = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "Hello World!\n"
    }
}
</pre>

As you can see, it is quite easy to write a simple Lambda function compared to a complete server.

To use serverless we will need to add another file to our project called `serverless.yml`

`touch serverless.yml`{{execute}}

and copy the following configuration:

<pre class="file" data-filename="project-red/serverless.yml" data-target="replace">
service: project-red

provider:
  name: aws
  runtime: nodejs12.x
  lambdaHashingVersion: 20201221

plugins:
  - serverless-plugin-typescript
  - serverless-offline

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: api/hello
          method: GET
          cors: true
</pre>

This says that we want to create a new service called `project-red` on `aws`.

We have one function called `hello` and we are telling serverless to look for a function called `hello` in our `handler` file to know what to do with it. We also specify a path `api/hello` which is the url path of the function. We also add the plugins we installed earlier here.

If we now run:

`serverless offline`{{execute}}

We can see that it is running at `http://localhost:3000/dev/api/hello`.

Try getting a response by running the following cURL command in a second terminal window:

`curl http://localhost:3000/dev/api/hello`{{execute T2}}

You should now see `Hello World!` printed to the terminal. In the next step we will add another Lambda that returns a secret message instead.