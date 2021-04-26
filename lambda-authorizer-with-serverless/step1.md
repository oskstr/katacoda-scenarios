## Setup
Let's start by getting our feet wet with AWS Lambdas in general. First we would just like to have a simple function that returns `"Hello World!`.

Think of a project name. Got it? I'll call mine `project-red` but you can choose whatever you like. Start by creating a new folder with that name and jump to it.

Type the following commands or run them by clicking the box:

```
mkdir project-red
cd project-red
```{{execute}}

Now that we are in the correct folder we can initialize the project with default settings:

`npm init -y`{{execute}}

Add `aws-lambda` as a dependency:

`npm install aws-lambda`{{execute}}

Install `serverless` globally:

`npm install -g serverless`{{execute}}

We are going to install the following developer dependencies:
- `typescript` - Compile our typescript code
- `serverless-plugin-typescript` - Automatic typescript compilation for serverless
- `serverless-offline` - Test code against emulated AWS Lambda and API Gateway
- `@types/aws-lambda` - Typechecking of types in `aws-lambda` package 

`npm install -D typescript serverless-plugin-typescript \
               serverless-offline @types/aws-lambda`{{execute}}


The installation might take a while but in the meantime you can start looking at the next step.


