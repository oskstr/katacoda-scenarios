This step should be fairly straight forward. What we will do is to add another Lambda function that we later will require to be authorized.

Start by adding the second Lambda to `handler.ts`:

<pre class="file" data-filename="project-red/handler.ts" data-target="replace">
import { APIGatewayProxyResult } from 'aws-lambda';

export const hello = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "Hello World!"
    }
}

export const secret = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "This is a super secret message that only authorized users should see!"
    }
}
</pre>

Then add the function to our `serverless.yml`:

<pre class="file" data-filename="project-red/serverless.yml" data-target="replace">
service: project-red

provider:
  name: aws
  runtime: nodejs12.x
  lambdaHashingVersion: 20201221

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: api/hello
          method: GET
          cors: true

  get-secret:
    handler: handler.secret
    events:
     - http:
          path: api/secret
          method: GET
          cors: true
    
plugins:
  - serverless-plugin-typescript
  - serverless-offline
</pre>

So far nothing special.