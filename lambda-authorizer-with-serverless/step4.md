There are many ways to handle authorization. Amazon even provides a way to generate API Keys directly. But that is not always an option. This might just be a small part of your system and you might want to handle authorization the same way here.

You could perhaps want to authenticate against som OAuth provider, use JWT tokens, or something completely different.

To show that anything would work, we will use [PokéAPI](https://pokeapi.co/) which is an API for Pokémon data. What if we just want to show our secret to people who can provide a Pokémon with the *Infiltrator* ability. 

### Policies
A policy is an object that specifies if the caller is allowed or denied to use certain resources. So what the Lambda Authorizer should return generally has this format:

<pre>
{
    principalId: "some-user-identifier",
    policyDocument: {
        "Version": "2012-10-17",
        "Statement": [
            {
            "Action": "execute-api:Invoke",
            "Effect": "Allow",
            "Resource": "arn:aws:execute-api:us-east-1:123456789012:ivdtdhp7b5/ESTestInvoke-stage/GET/"
            }
        ]
    }
}
</pre>

### Lambda Authorizer
First we will just install another dependency, `axios`, to make HTTP calls.

`npm install axios`{{execute interrupt T1}}


Then change `handler.ts` to:

<pre class="file" data-filename="project-red/handler.ts" data-target="replace">
import { 
    APIGatewayProxyResult, 
    APIGatewayAuthorizerResult,
    APIGatewayRequestAuthorizerEvent,
} from 'aws-lambda';
import axios from 'axios';

export const hello = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "Hello World!\n"
    }
}

export const secret = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "This is a super secret message that only authorized users should see!\n"
    }
}

export const auth = async (event: APIGatewayRequestAuthorizerEvent): Promise&lt;APIGatewayAuthorizerResult&gt; => {
  const { Authorization: token } = event.headers
  
  return axios.get(`https://pokeapi.co/api/v2/pokemon/${token}`)
      .then(({data}) => {
        if (data.abilities.some(({ability}) => ability.name === "infiltrator")) {
          return policy(true, token, event.methodArn)
        } else {
          return policy(false, token, event.methodArn)
        }})
      .catch(_ => {
        throw Error('Unauthorized')
      })
}


const policy = (allow: boolean, principalId: string, methodArn: string) => {
    return {
        principalId,
        policyDocument: {
            Version: '2012-10-17',
            Statement: [
                {
                    Action: 'execute-api:Invoke',
                    Effect: allow ? 'Allow' : 'Deny',
                    Resource: methodArn,
                },
            ]
        }
    }
}
</pre>

and change `serverless.yml` to:

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
          authorizer:
            name: auth
            type: request
            identitySource: method.request.header.Authorization
  auth:
    handler: handler.auth

plugins:
  - serverless-plugin-typescript
  - serverless-offline
</pre>


Restart the server again:

`serverless offline`{{execute interrupt T1}}

Now if we try to access the endpoints:

`curl http://localhost:3000/dev/api/hello`{{execute T2}}

We still see `Hello World!` as expected.

`curl http://localhost:3000/dev/api/secret`{{execute T2}}