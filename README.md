# MergeFiles
 Merge multiple Word/PDF files into one PDF file

Merge Documents

Overview
This lambda function combines the input word document file(DOCX) or PDF files into a single PDF file. Input is passed into the lambda as a JSON format. Functionality of this lambda function is divided into 4 parts.
Download all the input files from S3 bucket into the local repository. 
Check the extension of the file. If it’s a word file then convert it into a pdf file.
Now, at this point all the files are in PDF format. Call the Merge file function to combine all the files into a single  PDF file.
Once all the files have been successfully merged then create a pre signed URL of that file and return it in the response of the lambda function.


Implementation
Coding Language  : Python 3.7
Library/Tools used : VSCode, AWS Cloud9, PDFTron, AWS S3, AWS Lambda

PDFTron is an open source library which we can use to perform almost all the operations on a PDF file. It supports multiple languages like Node.js, Java, Python etc. I have used this library in merging docx/ pdf files into a single PDF.
In python it supports upto version <3.8.

Create a Lambda function, add the required roles to it from the Permission setting inside Configuration. Role likeAmazonS3FullAccess. 

After creating the lambda function setup the code in local.
Command to install the library: 
> python -m pip install boto3 -t .
> python -m pip install PDFNetPython3 -t .

To upload the code on Lambda, we have to zip all the libraries and code together. 
Command to zip the code and library into the current directory: 
> zip -r lambda_function.zip .
(Period means all)

To run and test the lambda function, cross check the runtime setting of lambda function.
In Handler the first parameter should be the file_name of the code file. Like for example my code file name is “lambda_function.py”, so lambda_function should come first. Then function name which invoked our lambda function and which takes the event and context as an input.
 

Input
{
  "files": [
    "a0aa0f2c-da69-4420-b5ea-bcd06b93ffb1/1617646540564/NJIT Intern Packet Spring 2021.pdf",
    "a0aa0f2c-da69-4420-b5ea-bcd06b93ffb1/1617646540564/a0aa0f2c-da69-4420-b5ea-bcd06b93ffb116176465405640.pdf",
    "a0aa0f2c-da69-4420-b5ea-bcd06b93ffb1/1617646540564/a0aa0f2c-da69-4420-b5ea-bcd06b93ffb116176465405641.vnd.openxmlformats-officedocument.wordprocessingml.document"
  ]
}

Output
{
  "statusCode": 200,
  "Pre-Signed-URL": "URL of the Final Merge File"
}


Issues/Error Occurred While Developments

Searching for the best library to implement the use case. After exploring a couple of libraries I got the PDFTron which is suitable for all use cases. But while installing it I got error like:
>python -m pip install PDFNetPython3
Could not find a version that satisfies the requirement PDFNetPython3
ERROR: No matching distribution found for PDFNetPython3

Reason:: IT only supports the python version upto 3.8 and I was using 3.9. SO I have to downgrade the version to install it locally.

Error while using the lib in AWS lambda.  
ModuleNotFoundError: No module named '_PDFNetPython'
File "/usr/lib64/python3.7/importlib/__init__.py", line 127, 
Reason:: I did not exactly find out the reason for this but I think its because of the OS difference. I am using windows and lambda support unix. 
When I used the Cloud 9, Import all the libraries and compressing and uploading the code from Cloud 9 the error got resolved and it get working.


In lambda when we download the files, we always have to download into the “tmp” folder always. I was not giving the “/tmp/” location when downloading the file into local and reading those from local. After correcting the path the code was working fine. s3.Bucket(BUCKET_NAME).download_file(KEY,'/tmp/' + KEY)

Also Check the permission which you have given for the lambda. I got error like 
[ERROR] OSError: [Errno 30] Read-only file system: 'Assignment1_Sol.pdf.CE332DEE'
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 130, in lambda_handler
    s3.Bucket(BUCKET_NAME).download_file(KEY, KEY)

Reason: I didn’t give the correct permission and role policy to the lambda.

TimeOut error while executing the lambda function.
Cross check the General configuration setting and increase the timeout in the lambda function. By default it’s set to 3 sec.Aslo give the memory as needed for your code.




Access denied while uploading the file to S3. Error like  S3UploadFailedError: Failed to upload . for that we have to add the permission and policy. https://aws.amazon.com/premiumsupport/knowledge-center/lambda-execution-role-s3-bucket/



