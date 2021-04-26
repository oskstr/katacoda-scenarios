## Step 2 - My first Lambda

Now we can create a file where our Lambda functions will exist. Use the following command:

`touch handler.ts`{{execute}}

Let's type the following code in `handler.ts` or simply press the "Copy to Editor" button:

<pre class="file" data-filename="project-red/handler.ts" data-target="replace">
import { APIGatewayProxyResult } from 'aws-lambda';

export const hello = async (): Promise&lt;APIGatewayProxyResult&gt; => {
    return {
        statusCode: 200,
        body: "Hello World!"
    }
}
</pre>

As you can see, it is quite easy to write a simple Lambda function compared to a complete server.
