This step should be fairly straight forward. What we will do is to add another Lambda function that we later will require to be authorized.

Start by adding the second Lambda to `handler.ts`:

<pre class="file" data-filename="project-red/handler.ts" data-target="append">
export const secret = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "This is a super secret message that only authorized users should see!\n"
    }
}
</pre>

Then add the function to our `serverless.yml`:

<pre class="file" data-filename="project-red/serverless.yml" data-target="append">
  get-secret:
    handler: handler.secret
    events:
      - http:
          path: api/secret
          method: GET
          cors: true
</pre>

So far nothing special. Restart the server and try to access the endpoints.

`serverless offline`{{execute interrupt T1}}

We should see that we have another function as well as another endpoint.

`curl http://localhost:3000/dev/api/hello`{{execute T2}}

We see "`Hello World!`", so far so good.

`curl http://localhost:3000/dev/api/secret`{{execute T2}}

😱 we seem to be leaking our secret. That's not good. In the next step we will finally add our Lambda Authorizer to protect this endpoint.