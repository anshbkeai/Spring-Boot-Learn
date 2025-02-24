# API Versioning in Spring Boot

Links  To  seee 
1. https://medium.com/@vivekkadiyanits/rest-api-versioning-in-spring-boot-application-7e47f0cbd8f4
2. https://www.postman.com/api-platform/api-versioning/
3. https://stackoverflow.com/questions/389169/best-practices-for-api-versioning

## Methods of API Versioning

1. **URL Path Versioning**  
   Example:  
http://localhost:8080/api/v1/docs
When scaling up, the version can be updated:  
http://localhost:8080/api/v2/docs


2. **Query Parameter Versioning**  
Example:  
http://localhost:8989/api/docs?version=1
. **Header Versioning**  
Example:

``` java @GetMapping(value = "/applicant", headers = {"x-api-version=2"})
public List<ApplicantRsponseVersion2> getApplicationURI2() {  
    ArrayList<ApplicantRsponseVersion2> applicant = new ArrayList<>();
    applicant.add(new ApplicantRsponseVersion2("ram", "34", "male"));
    return applicant;
} ''
Here, the API will respond only when the request contains the header:


x-api-version: 2

```
Media Type Versioning (Accept Header Versioning)
Example:
``` java

@GetMapping(value = "/applicant", produces = {"application/version2+json"})
public List<ApplicantRsponseVersion2> getApplicationURI2() {  
    ArrayList<ApplicantRsponseVersion2> applicant = new ArrayList<>();
    applicant.add(new ApplicantRsponseVersion2("ram", "34", "male"));
    return applicant;
}
The response will only be returned if the request's Accept header is set to:

Accept: application/version2+json
```

**Things yet to be Covered **
1,Rate limiting 
2.  auth using the API key