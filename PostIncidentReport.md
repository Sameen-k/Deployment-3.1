# Post-Incident Report

### Incident
A new hire was tasked with updating the URL shortener. The new hire committed version 2 of the application to the main branch. Which automatically triggered a build, test, and deploy to the production server, replacing version 1 of the application running on the server.
The URL shortener application was successfully deployed, however after it was updated, it was nonfunctional. When attempting to shorten the URL the application returned the following error message. 

<img width="1494" alt="Screen Shot 2023-09-20 at 8 33 05 PM" src="https://github.com/Sameen-k/Deployment-3.1/assets/128739962/49874c1c-f1c7-466b-a547-2d5fef9748db">

Because of the nature of the error occurring, it can be inferred that the issue is occurring in the application files as the application was successfully deployed.

### Initial Steps
The first plan of action was to rollback the application back to version 1 which was in working condition before the employee updated it using Git. 
First pulled up the commit history by using the following command

``git log --oneline``

![Screen Shot 2023-09-22 at 9 03 42 PM](https://github.com/Sameen-k/Deployment-3.1/assets/128739962/f31df4ce-9316-4e7c-a05d-8a94852fd6f3)

This shows each commit made as well as the commit ID, which is important for the next command. We can see in the logs the version 1 commit as well as the version 2 commit.
to revert back to version 1, we can use the following command 

``git checkout <commit ID> .``

in this case: 

``git checkout 4707109 .``

This command will put you in the version 1 of the application. Afterwards you just have to add ALL of the files to the commit by using the following command

``git add .``

and then make it a commit and leave a comment that you're reverting back to version 1 due to an issue present in version 2

``git commit -m "Rolling back to version 1 <commit-id>"``

All that's left is to push this version to remote to save the changes you just made. This is a bit complicated since the version history is attached to a different repo that we cloned from. 
So when we do ``git push`` we get the following error:

<img width="683" alt="Screen Shot 2023-09-20 at 8 34 13 PM" src="https://github.com/Sameen-k/Deployment-3.1/assets/128739962/de9a42fe-9870-4d41-b93e-bc69d3a52317">

To solve this error, we can follow the directions Git provides in the error it responded with. We can do a git pull: ``git pull origin main``
This gives us the following rejection:

<img width="695" alt="Screen Shot 2023-09-22 at 1 21 17 PM" src="https://github.com/Sameen-k/Deployment-3.1/assets/128739962/b56db4c3-9a70-4b5e-a523-764448852ae5">

In this case, we can go with the first option Git presents which is the ``git config pull.rebase false``
This command is to merge the conflicting branches into the main. After this you can finally proceed with ``git push`` and you will have successfully reverted back to version 1 of the application 

This can all be done in a matter of minutes and is the first priority so as to not allow a dysfunctional application to stay running.

### Resolution
After reverting back to the functional version 1 of the application, we can now focus on addressing the issue the employee may have caused. As previously stated, the error seemed to be related to the application code itself.
Using this information we can narrow down the search for the error to the application script. 
To more easily be pointed in the right direction, we can read the AWS Elastic Beanstalk logs for the environment. We can scroll down to the error logs and see what errors are present:

![Screen Shot 2023-09-22 at 10 32 06 PM](https://github.com/Sameen-k/Deployment-3.1/assets/128739962/b9fffb94-809b-4dc0-a01d-bcf3a540af32)

the error states that in Line 339, in Loads, the JSON object must be a string. We can go to the script in GitHub of version 2 and view the line where the issue is:

#### Version 2:
![Screen Shot 2023-09-22 at 7 06 14 PM](https://github.com/Sameen-k/Deployment-3.1/assets/128739962/e5341e3f-d04f-4c6c-98f3-de4915a84114)


Using GitHub we can also carefully examine the differences between the code in version 1 vs. the code in version 2:

#### Version 1:
![Screen Shot 2023-09-22 at 7 06 00 PM](https://github.com/Sameen-k/Deployment-3.1/assets/128739962/347d6501-7fcc-4524-8cf1-d444ef770dc9)


Looking at the code from version 1 and 2 side by side, we can note that there's a difference between the command structure pertaining to the JSON object. Simply put, Version 1 has ``JSON.load`` and version 2 has ``JSON.loads``
After some research, the JSON.load structure is used to read the JSON from a document/file whereas the JSON.loads structure is used to convert a JSON string into a Python dictionary which is not applicable. The JSON file needs to be read in order for this code to work so that's why the issue occurred. Of course, it can easily be resolved by changing the JSON.loads to JSON.load

### Prevention
This issue could have been more efficiently avoided if the new employee had sufficient supervision for any changes they were to make to the application and in any case they may need some access restrictions to prevent them from making any unauthorized or unsupervised changes.
