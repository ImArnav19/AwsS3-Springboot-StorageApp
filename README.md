**AwsS3-Springboot-StorageApp**

Project Description 📦🚀
Spring Boot Application: Built with Spring Boot for a robust and scalable backend.
AWS S3 Integration: Seamlessly integrates with AWS S3 for efficient and secure storage.
RESTful API: Provides a simple and clear RESTful API for easy interaction.
CRUD Operations: Supports Create, Read, Update, Delete operations for S3 objects.
Exception Handling: Robust error handling and logging for smoother operations.
Security: Implements security best practices to protect your data.


Features 🌟
Upload Files: Upload any file type to AWS S3 with ease.
Download Files: Securely download files from AWS S3.
List Files: View all files stored in your S3 bucket.
Delete Files: Remove files from your S3 storage as needed.

Technologies Used 🛠️
Java: Core language used for development.
Spring Boot: Framework for building the application.
AWS SDK: AWS SDK for Java to interact with S3.
Maven: Dependency management and build tool.

Getting Started 🚀
Clone the Repository: git clone https://github.com/ImArnav19/AwsS3-Springboot-StorageApp.git
Navigate to Directory: cd AwsS3-Springboot-StorageApp
Configure AWS Credentials: Set your AWS credentials and S3 bucket name in application.properties.
Build the Project: mvn clean install
Run the Application: mvn spring-boot:run

Let's Follow a Step By Step Process To Help Understand and Enjoy

1. Load the Postgres DB and create Database container

![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/2fe4580e-b693-4129-b7c2-e9f2445a22c4)

-> Database Customer configured with spring Application
Having the React Code Ready with Credentials

![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/ee90b5ff-8e6d-491d-aaa0-533ac5a6aad1)

-> Now we must setup AWS SDK , it’s the main Api for java withAWS

Dependency :
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.20.26</version>
    <exclusions>
        <exclusion>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>netty-nio-client</artifactId>
        </exclusion>
        <exclusion>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>apache-client</artifactId>
        </exclusion>
    </exclusions>
</dependency>


-> Create IAM user with full access


![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/bf4de61b-8240-4cd8-bf41-4186fec123c1)

-> Creating Access keys so we get the Access_key and secret_access_key


![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/6b3699ee-142b-4c87-bfbb-f2a36d108d91)


-> now its time to start S3
Env variables are the ones which changes
S3 build client 

S3 build client 
package com.amigoscode.auth.s3;


import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;

@Configuration
public class S3Config {

    @Value("${aws.region}")
    private String awsRegion;

    @Value("${aws.s3.mock}")
    private boolean mock;

    @Bean
    public S3Client s3Client() {
        if (mock) {
            return new FakeS3();
        }
        return S3Client.builder()
                .region(Region.of(awsRegion))
                .build();
    }

}

Now we must make our S3 services to add and put object

package com.amigoscode.auth.s3;


import org.springframework.stereotype.Service;
import software.amazon.awssdk.core.ResponseInputStream;
import software.amazon.awssdk.core.sync.RequestBody;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.GetObjectRequest;
import software.amazon.awssdk.services.s3.model.GetObjectResponse;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;

import java.io.IOException;

@Service
public class S3Service {

    private final S3Client s3;

    public S3Service(S3Client s3) {
        this.s3 = s3;
    }

    public void putObject(String bucketName, String key, byte[] file) {
        PutObjectRequest objectRequest = PutObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();
        s3.putObject(objectRequest, RequestBody.fromBytes(file));
    }

    public byte[] getObject(String bucketName, String key) {
        GetObjectRequest getObjectRequest = GetObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();

        ResponseInputStream<GetObjectResponse> res = s3.getObject(getObjectRequest);

        try {
            return res.readAllBytes();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

    }
}

Bucket for operation : 

![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/10bfa1cf-d129-4bf0-8b91-8140c4fd8c60)

Now we can have a test module so see if connections is working well
We can store more such files 


![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/81b6208c-1dc5-4af6-8b58-4fd279e3796f)
![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/c516f788-48be-40f5-bc8e-a630db42624a)


-> Now its time we create S3 bucket class for storing

These routes makes sure the hanling is done by Customer Service 
we have used different types of storing media files for better connectivity

For Customer Service the storage is profile-images/{custId}/{profilePic_Id}
public void uploadCustomerProfileImage(Integer customerId,
                                       MultipartFile file) {
    checkIfCustomerExistsOrThrow(customerId);
    String profileImageId = UUID.randomUUID().toString();
    try {
        s3Service.putObject(
                s3Buckets.getCustomer(),
                "profile-images/%s/%s".formatted(customerId, profileImageId),
                file.getBytes()
        );
    } catch (IOException e) {
        throw new RuntimeException("failed to upload profile image", e);
    }
    customerDao.updateCustomerProfileImageId(profileImageId, customerId);
}


-> Now its time we alter the Database and make sure add Profile_id 
in db_properties : 

ALTER TABLE customer
ADD COLUMN profile_image_id VARCHAR(36);

ALTER TABLE customer
ADD CONSTRAINT profile_image_id_unique UNIQUE (profile_image_id);

With this we must ensure all the other DTO,DAO mapper classes that we are using must be altered to cater the requests

After which we test the module if in DB its storing


![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/839afefb-4590-42cd-8877-3edf7d45154e)


Running Perfect!!

Setting up the react api routes and designs


And After that here are the results : 

![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/360fc1d6-45d5-4351-a0b4-2c3291aca668)

![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/172faa1a-5c86-4667-8dcd-83abeb142234)

![image](https://github.com/ImArnav19/AwsS3-Springboot-StorageApp/assets/117253613/048431d6-9173-4ea4-89fa-356fb8d30017)



