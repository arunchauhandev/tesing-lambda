Note: Deploy node function to lambda steps

Step 1: Create Project Structure like this

    my-lambda-app/
    ├── src/
    │   └── index.js
    ├── package.json
    ├── package-lock.json
    └── .github/workflows/deploy.yml

Step 2: Create a function

    exports.handler = async (event) => {
        const response = {
            statusCode: 200,
            body: JSON.stringify('Hello from Lambda!'),
        };
        return response;
    };

Step 3: use provided YAML

    name: Deploy to AWS Lambda

    on:
    push:
        branches: [main]
    pull_request:
        branches: [main]

    jobs:
    deploy:
        runs-on: ubuntu-latest

        steps:
        - uses: actions/checkout@v3

        - name: Setup Node.js
            uses: actions/setup-node@v3
            with:
            node-version: "18"
            cache: "npm"

        - name: Install dependencies
            run: npm ci --production

        - name: Create deployment package
            run: zip -r function.zip src/ node_modules/

        - name: Configure AWS credentials
            uses: aws-actions/configure-aws-credentials@v2
            with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: ap-south-1

        - name: Deploy to Lambda
            run: |
            # Try to update existing function first
            if aws lambda update-function-code \
                --function-name my-lambda-function \
                --zip-file fileb://function.zip 2>/dev/null; then
                echo "Function updated successfully"
            else
                echo "Update failed, function might not exist. Consider creating it first."
                exit 1
            fi

Step 4: setup AWS and requirements

    Create an IAM user with these policies:

    AWSLambda_FullAccess
    S3FullAccess (for deployment artifacts)
