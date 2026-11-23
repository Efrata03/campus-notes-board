# Break it yourself: troubleshooting notes

Each fault was tested separately. After each test, the original configuration was restored and verified.

## Fault 1: Database security group

- **Original configuration:** The db-sg security group allowed MySQL TCP port 3306 from app-sg.
- **What I changed:** Temporarily removed the MySQL inbound rule.
- **Symptom (what I saw, and where):** The application health page showed app: ok, database: error (OperationalError 2003), and s3: ok.
- **Possible causes I considered:** Incorrect database endpoint, database availability, network connectivity, or a security group rule blocking MySQL.
- **How I verified the actual cause:** The health check showed that the application and S3 were working, but the database connection failed after the inbound rule was removed.
- **Fix:** Restored the MySQL TCP 3306 inbound rule with source app-sg in db-sg.
- **What I learned:** Security groups control network access between AWS resources. The database must allow connections from the application security group.

## Fault 2: S3 permission

- **Original configuration:** The notes-ec2-role policy allowed s3:ListBucket, s3:GetObject, and s3:PutObject for the required bucket and notes objects.
- **What I changed:** Temporarily removed s3:GetObject from the inline policy campus-notes-s3-policy.
- **Symptom (what I saw, and where):** Downloading a PDF showed "Storage unavailable. The PDF could not be retrieved."
- **Possible causes I considered:** Incorrect S3 object key, missing object, bucket access restrictions, or missing IAM permissions.
- **How I verified the actual cause:** Checked the application logs using sudo journalctl -u notes -n 50 --no-pager. The logs showed AccessDenied and stated that no identity-based policy allowed s3:GetObject.
- **Fix:** Restored s3:GetObject in the ReadWriteNotesObjects statement of campus-notes-s3-policy.
- **Verification:** The same PDF downloaded successfully after the permission was restored.
- **What I learned:** IAM permissions determine which AWS actions an application role can perform. Downloading an S3 object requires s3:GetObject.

## Fault 3: Public route

- **Original configuration:** The public-rt route table contained a local route for 10.42.0.0/16 and a default route 0.0.0.0/0 pointing to the Internet Gateway igw-00c711eb4f84760f1.
- **What I changed:** Temporarily removed the default route 0.0.0.0/0.
- **Symptom (what I saw, and where):** The public Campus Notes Board website could not be reached.
- **Possible causes I considered:** Application failure, web server failure, or missing network routing.
- **How I verified the actual cause:** The website became unreachable after the default Internet Gateway route was removed.
- **Fix:** Restored the 0.0.0.0/0 route with target igw-00c711eb4f84760f1.
- **Verification:** The website opened successfully after the route was restored.
- **What I learned:** A public subnet needs a default route to an Internet Gateway for internet connectivity.

## Conclusion

All three troubleshooting exercises were performed separately. Each original configuration was restored, and the application was verified after restoration.
