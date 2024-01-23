1. 1. 1. First we created an S3 bucket on aws, after creating a free-tier account. We upload our website files to it, and make all objects/files publicly readable. the ones which we want to serve as our static website. Then we enable the static hosting in the properties of our bucket. This gives us an endpoint to reach that website from any browser.

      2. S3 - simple storage service, key-blob store, reliable, consistent. It's not a file system.\
         AWS policies define access control for:

      \- users

      \- other AWS services

      \- resources

      3. We are going to be uploading many files, and so that we don’t have to go through the process of changing the access control to public everytime, we set a policy on our s3 bucket that everything is going to be publicly accessible for our website.\
         ![](https://lh7-us.googleusercontent.com/Que-lQi0KJNMj8LlT_vcUlEa3VHmUIHYNsQoRLNGgp6l9ABkwlfvnfGzdHs2poAxJd_2BuDB5HoO9Vle_L7xPE_zpRbHSPQEvuMII95Nnnd_hwfR5eeo8E01OC9SEy4idWgsDrX3IWcNWLP79aig4mg)

      4.  So we change the “Effect”  property in our statement tag in our policy to “Allow”. And a “ \* ” wildcard in the “Principal” property. To denote anyone has access to the files now. “Action” states what action is the policy to be applied on, i.e. in our case - retrieve information from s3, then we state the resource itself.

      5. AWS lambda service - executes our function in the cloud as many times as we need. And we have AWS IAM -\
         IAM – Identity and Access Management which lets us manage - users/groups, roles, policies, identity providers, user account settings, encryption keys.

      6. We need to think about roles and policies, anytime we have 2 aws services communicating with each other. We need to define the access those services have with each other. So now we will create a role and a policy so that our lambda function can talk to the s3 bucket.\
         We created a policy -> chat-s3-access, and will assign it to our newly created role - “chat-lambda-data”. We also add the - “AWSLambdaBasicExecutionRole” policy to it.

      7. We now create a lambda function - “Chat-API-proxy”, add our custom role to it, and deploy it. The role has permissions to edit the function. Our Lambda function fetches the JSON file ('data/conversations.json') from an S3 bucket and returns the content as a JSON response. It handles both successful and error scenarios and includes CORS headers for cross-origin support.

      8. We will create an api for our chat app that will execute the lambda function. There are many ways to trigger the lambda functions. - API Gateway, AWS IoT, Alexa Skills, Alexa Smart Home, CloudWatch Events, CloudWatch Logs, CodeCommit, Cognito Sync, DynamoDB, Kinesis, S3, SNS

      9. The API gateway can be used in 2 different ways-\
         1\. Proxy mode - all requests to the configured resource will be sent on to the lambda function without any modification. Basically takes an http request from our webapp and turns it into an event for lambda\
         2\. Mapping mode - maps the input into something more specific to our lambda function

      10. Next we will build a REST API in our API Gateway, and ADD a proxy resource in it. - {proxy+}, basically catches any request to the API. For example, if a request is made to “/myapi/resource1”, the {proxy+} parameter would be populated with resource1. If the request is to “/myapi/resource2/subresource”, {proxy+} would be “resource2/subresource”.\
          We integrate our lambda function to it.\
          We enable CORS on our api. By enabling CORS on your API, you allow web applications from different origins to make requests to your API, subject to the configured restrictions.\
          When you enable CORS on your API, you are explicitly allowing cross-origin requests. This is important in scenarios where a web application hosted on one domain needs to make HTTP requests to an API hosted on a different domain. Without CORS, the browser would block such requests, and the web application would not be able to fetch data or interact with the API.\
          So to use resources from other sites in our site

      11. We now deploy our api and it gives us an endpoint. Now after adding “/conversations” to the endpoint, we get our conversations data back. So we are going through the api gateway which in turn is calling the lambda function that’s retrieving the json data from the s3 and sending the data back to us through the api gateway. We update our code to do so. And upload it to s3.\
          The updated lambda handler function extracts the proxy path from the url. And based on the path sets the key used to GET the object from s3.\
          Similarly we make changes to the code to retrieve individual conversations, through the api gateway.\
          On the client side, we get the ChatApp.apiEndpoint, the url and append “/conversations” to get the list of chats. And '/conversations/#\<chat\_id>' to get individual chats.

      12. We will move to DynamoDB which is a nosql database, extremely scalable but no ACID principles adhered\
          ![](https://lh7-us.googleusercontent.com/aMAHxIGMLB7hYlWvXuataiCFpjGEnaf1DdiB0NtkApHHFadVGsEm1rOzp1DP-Vn1YgL0gErXCj42QWPedV6Q0Z-sJQvE8yhAZ-PfvkowlsIQKICoUn3yE_hfIBbgDtoIyJAmwM0P1KHxr-pqx3dowHA)\
          Types of attributes -\
          ![](https://lh7-us.googleusercontent.com/YmnTSsoORtj1nDa5OgV_lkuZrN2IL7myI7vFSDeLSrOoc0w_Zi67M-bsB0oCzqd-BtALUcS6AKcKjAuGofH40vgaYH5OHBZCFJmNoNYo3zRON-wRsxjtaJmWzTG1dHKPfXcD_i7p6B-N6Fkw3_bkPKk)\
          Retrieving items -\
          ![](https://lh7-us.googleusercontent.com/4JNmbrLrtBABp1kJaCjdMx6EEVEatesDdeIMu-q514IDf8-dbEpyMd8aXpjKQd7ew0moMtIlXahaj4BS11A1O4pyvrHGU_DCOfd9behQBXqN2M-Qno6yNCdjzIEBeK6PuFwaMk0dcQb1Iosq2WKP-C8)

      13.  DynamoDB has - Partition Key: The partition key is the primary attribute used to determine the physical storage location of an item within the DynamoDB table.\
          And , Sort Key (Optional): The sort key is also known as the range key. It is optional and used to further organize items within a partition.\
          We have disabled the auto-scaling which is a feature that automatically adjusts the read and write capacity of a DynamoDB table in response to changes in application traffic. With Auto Scaling, you can dynamically adapt your provisioned throughput capacity to handle varying workloads without manual intervention. Right now the provisioned capacity units are set to 1.

      14. So right now we have 2 tables in dynamoDB -\
          Chat-conversations - just tells us who’s in what conversations\
          Chat-messages (we created a secondary index for this one)\
          We create a policy for dynamoDB with these actions- dynamodb:BatchWriteItem, dynamodb:PutItem, and dynamodb:Query. These actions are related to writing and querying items in DynamoDB. And we add this policy to our chat-lambda-data role

      15. Now, in our Lambda function, we query the dynamoDB with the conversationId extracted from the request. paginateQuery allows us to process discrete chunks of data back from the dynamoDB. In the query, we specify the tableName, the attributes we want to retrieve in projectionExpression, which is like a SELECT statement, (reminder - there’s a 1MB limit for each request).\
          This lambda function supports two main paths: 'conversations' and 'conversations/{id}'\
          For the ‘conversations’ path-\
          It retrieves conversation IDs where the username is 'Student.' Loads the last message timestamp and participants for each conversation. Assembles this and returns as a json.\
          And for the 'conversations/{id}' path:\
          Action is based on whether the HTTP method is GET or POST.\
          for GET - Query the "Chat-Messages" table for messages with the specified conversation ID and Load conversation details, including participants and the timestamp of the last message.\
          For POST - Insert a new message into the "Chat-Messages" table with the specified conversation ID, message content, sender ('Student'), and timestamp.\
          —------------------------------XXX—-------------------------------------XXX—----------------------------------

      16. Now we will leave proxy mode and utilize the complete potential of the API gateway.\
          A model in the api gateway defines input and output types and shapes. For the request data, we will define the body shape (only applies to write methods of the requests). And body shape for response data as well. And we will use JSON Schema to define it.

      17. We will take a look at the request flow, from the client to our lambda function. The request flow is broken into 2 pieces -\
          1\. Method request - deals with the shape of the incoming request. Validation, caching, authorization, what input does it need ?. Once the method request has handled the actual connection to the client and the parsing of the input, the integration request lives between that and the implementation which is the lambda function in our case.\
          For authorizing incoming requests we have - AWS IAM, API key, Cognito (the authorizer looks at the specified header for a cognito token that it can verify. Since that token is specific to the user since they have logged in, it's not something that can be used to impersonate anybody else), Custom\
          2\. Integration request

      18. Similarly the response flow is also divided into 2 parts - method response (defines the types of responses that the user should be able to expect) and integration response.

      19. So first we create 2 resources in our API - conversations and conversations/{id}. And get started with our Lambda function - first is “chat-conversations-GET” which we will integrate with our API while creating a GET method in it.\
          We create a method for our method response in GET method of the API. and on test, we get this response body -\
          ![](https://lh7-us.googleusercontent.com/SkzK_uf3zwez_khvkdu7j4vdsgVdUUZY5Wx95HAVcAismAGCaSjHszn8eRGcgCIBtEF4efWWJmZm4dst-nfYJkiEI1H2eIdMeGI2CQ-JjSlh9uDLos4rViLfx-As-y2ULx4sn5oVuCa_WYknm7fKqK8)\
          Given our json-schema was like this-\
          ![](https://lh7-us.googleusercontent.com/bQFnwBVmGmqeggKbxxTWw7PagoMD3SVKpxEOd3GG_cFsNW36ULERViv7iE36JEqBvVW3Vtm3lXusO1FoYwfThQMMnlXDyRvHbRZMd7HuUJXPkk62LZshBAN3QJTS41I3v-PYWwZmdv9BAQhqFdIaslg)

      ****

      27\.

      We will add the cognito authorizer to all methods of our api resources. Also we have the mapping templates for the integration requests which allows us to define how the incoming request payload, headers, and parameters should be transformed before being sent to the backend.
