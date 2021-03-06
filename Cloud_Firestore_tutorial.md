# What is Cloud Firestore

Cloud Firestore(beta) is a scalable NoSQL database for web, mobile, etc. Cloud Firestire will keep our data in sync aross client apps and offers offline supportso that we can create responsive apps regredaless of our internet connectivity.

TWO GOOD READINGS:

https://cloud.google.com/firestore/

https://firebase.google.com/docs/firestore/


Get Started 

1. [Go to Firebase](https://firebase.google.com/)
* click get started
* add a new project

2. Go to the DATABASE section and select Get Started button for Cloud Firestore

3. Select **Test Mode** for your Cloud Firestore Security Rule. At later stage you might want to switch it into **Locked Mode**. Then click enable.


Set up the environment

1. Add Firebase Admin SDK:

* Using Gradle: 

ADD:

compile 'com.google.firebase:firebase-admin:6.4.0'

* Using Maven: 

In dependency ADD:

```
 <!-- https://mvnrepository.com/artifact/com.google.firebase/firebase-admin -->
 <dependency>
    <groupId>com.google.firebase</groupId>
    <artifactId>firebase-admin</artifactId>
    <version>6.5.0</version>
 </dependency>
```

## Initialize Cloud Firestore

1.Create and set up new service account on [GCP Console](https://console.cloud.google.com/)
Use the same email address that you used for creating the new project on the Firebase. You should be able to find the exsiting project.

* open **IAM & Admin** page in the GCP Console, select your porject.
* In the left nav, click **Service accounts**
* Click **MORE and dots** (it is on the same row as the email account and located at the far right side) button and then click **Create Key** and select .json. 
* Download the .json file and rename it into **credentials.json**. 
* In the `.gitignore` add `credentials.json`

You do not want to upload the `credentials.json` file into github repo.
It contains private and sensitive information and you don't want other people to see it. 

2. Add `FIREBASE_JSON` variable

* create a `setup.sh` file to set the environment variable `FIREBASE_JSON`:

   ```
   export FIREBASE_JSON=`cat credentials.json`
   ```

3. Add this method inside your Java code to read the Firebase credentials 
   from the `FIREBASE_JSON` environment variable:

```
  public static String getFireBaseCredentials() {
        ProcessBuilder processBuilder = new ProcessBuilder();	
        if (processBuilder.environment().get("FIREBASE_JSON") != null) {
            return processBuilder.environment().get("FIREBASE_JSON");
        }
	throw new RuntimeException("no FireBase Credential found.");
    }
```

Finally inside the main function add this code which takes the credentials from that method,
instead of reading them from a file on the disk.  That's important because the only way to 
securely get the credentials into a running Heroku app is via an environment variable.

```
import com.google.auth.oauth2.GoogleCredentials;
import com.google.cloud.firestore.Firestore;
import com.google.firebase.FirebaseApp;
import com.google.firebase.FirebaseOptions;
```
...

```
   InputStream serviceAccount = new ByteArrayInputStream(getFireBaseCredentials().getBytes("UTF-8"));
   GoogleCredentials credentials = GoogleCredentials.fromStream(serviceAccount);
   FirebaseOptions options = new FirebaseOptions.Builder()	
    	.setCredentials(credentials)	
    	.build();	
   FirebaseApp.initializeApp(options);
   Firestore db = FirestoreClient.getFirestore();
```

## Add data and Read data
* add

```
DocumentReference docRef = db.collection("users").document("students");
Map<String, Object> data = new HashMap<>();
data.put("first", "XXX");
data.put("last", "ZZZ");
data.put("born",1997);
ApiFuture<WriteResult> result = docRef.set(data);
```

* read

```
ApiFuture<QuerySnapshot> query = db.collection("users").get();	
QuerySnapshot querySnapshot = query.get();
List<QueryDocumentSnapshot> documents = querySnapshot.getDocuments();	
for (QueryDocumentSnapshot document : documents) {
  System.out.println("User: " + document.getId());  
  System.out.println("First: " + document.getString("first"));  
  if (document.contains("middle")) {
  
    System.out.println("Middle: " + document.getString("middle"));
  }
  System.out.println("Last: " + document.getString("last"));  
  System.out.println("Born: " + document.getLong("born"));
}
```

## We also need to add the vriable `FIREBASE_JSON` inside Heroku

1. Go to heroku console and add the variable `FIREBASE_JSON` and then copy and paste the content insde crendentials.json into the `FIREBASE_JSON`

2. Or you can use this, provided you've set the current app

```
heroku config:set FIREBASE_JSON=`cat credentials.json`
```

If it complains about not knowing what the value of APP is, try:

```
heroku config:set FIREBASE_JSON=`cat credentials.json` --app appname
```

## Reference

* <https://firebase.google.com/docs/firestore/quickstart>

* <https://cloud.google.com/docs/authentication/production#auth-cloud-implicit-java>


