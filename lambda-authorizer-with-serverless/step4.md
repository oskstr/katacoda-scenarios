There are many ways to handle authorization. Amazon even provides a way to generate API Keys directly. But that is not always an option. This might just be a small part of your system and you might want to handle authorization the same way here.

You could perhaps want to authenticate against some OAuth provider, use JWT tokens, or something completely different.

To show that anything would work, we will use [PokéAPI](https://pokeapi.co/) which is an API for Pokémon data. What if we just want to show our secret to people who can provide a Pokémon with the *Infiltrator* ability. 

### Policies
A policy is an object that specifies if the caller is allowed or denied to use certain resources. So what the Lambda Authorizer should return generally has this format:

<pre class="file">
{
    "principalId": "some-user-identifier",
    "policyDocument": {
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

Let's see what happens with some different calls:
<pre class="file">
1. curl http://localhost:3000/dev/api/hello
Hello World!

2. curl http://localhost:3000/dev/api/secret
{"statusCode":401,"error":"Unauthorized","message":"Unauthorized"}

3. curl http://localhost:3000/dev/api/secret -H "Authorization: agumon"
{"statusCode":401,"error":"Unauthorized","message":"Unauthorized"}

4. curl http://localhost:3000/dev/api/secret -H "Authorization: bulbasaur"
{"statusCode":403,"error":"Forbidden","message":"User is not authorized to access this resource"}

5. curl http://localhost:3000/dev/api/secret -H "Authorization: zubat"
This is a super secret message that only authorized users should see!
</pre>

Okay, so what happened there? 

1. `/hello` still works like before, as expected.
2. The API Gateway will automatically deny this request since we didn't even provide an `Authorization` header.
3. We won't find any Pokémon called Agumon - since that is a Digimon character.
4. Still an error message but a different one. `403` means that the authorizer understood the request and found the "user" but it doesn't have the correct permissions. The correct permission would be the *Infiltrator* ability but Bulbasaur only has *Overgrow* and *Chlorophyll*. 
5. Zubat does have the *Infiltrator* ability 🎉


Nice! Our authorizer works. That's a win in my book! But its still only running locally. In the next step we will deploy our service to AWS using serverless.