# AI Code Assistant

## Overview
Sometimes the code files I am looking at for open-source development are very complex to understand because of the use of libraries that I might not have used or seen before, or because of poor code writing practises. The objective of making this tool is to find or analyse the files where a certain functionality exists, and answer code related questions.

## Architecture Diagram
![Milestone 2 architecture diagram](https://github.com/user-attachments/assets/a6adef26-5103-4647-aaba-2ae08dc03428)

## UML Diagram
![Milestone 2 UML diagram](https://github.com/user-attachments/assets/e83d2073-e58c-427d-9db9-bb171adf97de)

## Process Flow
I.	User first access the website through Amazon CloudFront.

II.	When the user lands on the website, the browser initiates the connection to web socket so that the responses could be retrieved later on.

III.	Once the connection is open, a unique user ID is generated and wrapped in a message. This message is sent to the web socket.

IV.	This causes a lambda function to trigger. The lambda function will then extract the user ID from the message, and the connection ID from its invocation metadata, and stores them in the DynamoDB.

V.	Now user either uploads the zip file of the code, or enters the corresponding Github URL, and sends a request to API Getway.

VI.	API Gateway triggers a lambda function that will process these inputs. If the user has uploaded zip file of the code, it will generate a presigned URL and send that URL to the front-end. If the user has uploaded Github URL, the URL along with the user ID will be passed to the Amazon SQS. Once the user uploads the file using the presigned URL, the file path will be passed to the same Amazon SQS.

VII.	EC2 instance managed by Elastic Beanstalk will poll the messages from the queue and start processing the uploaded files. It will embed the files in chunks using CodeBERT model and store the vectors in FAISS index, the metadata in another S3 bucket. 

VIII.	Once the processing is finished, it will query the DynamoDB for the connection ID using the user ID to notify the front-end using the web socket.

IX.	The front-end will allow user to input query.

X.	Once the query is entered, it will be sent as a message along with the user ID to the web socket, which will trigger a Lambda function. 

XI.	The lambda function will put this message in a queue. EC2 will poll the message and starts processing the query.

XII.	The query will be embedded using CodeBERT. Then the program will retrieve vectors stored in FAISS index and search for the top – k similar embeddings / vectors that match the query vector.

XIII.	Once the matches have been found, they will be augmented into the user prompt and passed to the AI model deployed (Code Llama / Meta Llama / CodeBERT) on Amazon Sagemaker.

XIV.	Amazon Sagemaker will make use of augmented context into the prompt and return a response to the EC2 instance.

XV.	EC2 instance will send the response in chunks to the front-end of the website through the web socket.

XVI.	The user will be able to see the responses as they are received from the back-end.

## Lessons learned and Future Improvements

Even though the model works reasonably well, if the embeddings aren’t good enough and if the code chunks aren’t properly embedded, the generated response could become irrelevant. In future, a better embedding model for query, and a better method for chunking user data could be used so that the context is relevant to the query and the LLM can answer the question. Currently, each file has its own chunk, and the embeddings aren’t as close to the chunk’s embedding as required for better performance. This causes the top_k result to vary, and therefore the results.


