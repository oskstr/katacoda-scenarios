## Step 1
Let's start by getting our feet wet with AWS Lambdas in general. First we would just like to have a simple function that returns `"Hello World!`.

Think of a project name. Got it? I'll call mine `project-red` but you can choose whatever you like. Start by creating a new folder with that name and jump to it.

Type the following commands or just click on them:

`mkdir project-red`{{execute}}

`cd project-red`{{execute}}

Now that we are in the correct folder we can initialize the project with default settings:

`npm init -y`{{execute}}

Add `aws-lambda` as a dependency:

`npm install aws-lambda`{{execute}}

We are going to install the following developer dependencies:
- `typescript` - To compile our typescript code
- `serverless` - Deploy using serverless framework
- `serverless-plugin-typescript` - Automatic typescript compilation for serverless
- `serverless-offline` - Test code against emulated AWS Lambda and API Gateway
- `@types/aws-lambda` - Typechecking of types in `aws-lambda` package 

`npm install -D typescript serverless serverless-plugin-typescript \
               serverless-offline @types/aws-lambda`{{execute}}

Now we can create a file where our Lambda functions will exist. Use the following command:

`touch handler.ts`{{execute}}

Let's type the following code in `handler.ts` or simply press the "Copy to Editor" button:

<pre class="file" data-filename="handler.ts" data-target="replace">
import { APIGatewayProxyResult } from 'aws-lambda';

export const hello = async (): Promise<APIGatewayProxyResult> => {
    return {
        statusCode: 200,
        body: "Hello World!"
    }
}
</pre>

As you can see, it is quite easy to write a simple Lambda function compared to a complete server.

