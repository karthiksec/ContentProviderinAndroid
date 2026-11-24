# Ex.No:3 Create Your Own Content Providers to get Contacts details
## AIM:

To create your own content providers to get contacts details using Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min. required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as “ex.no.3″ and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Get contacts details and Display details give in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
Program to print the text create your own content providers to get contacts details.
Developed by: KARTHIK G
Registeration Number : 212223220043
```
## activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Contact Details"
        android:textSize="28sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginTop="30dp"
        android:layout_marginBottom="20dp" />

    <Button
        android:id="@+id/btnLoadContacts"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Load Contacts"
        android:layout_marginBottom="20dp" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <TextView
            android:id="@+id/txtContacts"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:padding="10dp" />

    </ScrollView>

</LinearLayout>
```

## AndroidManifest.xml
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <uses-permission android:name="android.permission.READ_CONTACTS" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.ContentProvider">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```
## MainActivity.java
```
package com.example.contentprovider;

import android.Manifest;
import android.content.ContentProviderOperation;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.provider.ContactsContract;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    private static final int PERMISSION_REQUEST_CODE = 100;
    private static final String TAG = "ContactProviderLog";

    private Button btnViewContacts;
    private TextView tvStatus;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnViewContacts = findViewById(R.id.btnViewContacts);
        tvStatus = findViewById(R.id.tvStatus);

        btnViewContacts.setOnClickListener(v -> checkPermissionAndFetchContacts());
    }

    private void checkPermissionAndFetchContacts() {
        boolean readPermission = ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS) == PackageManager.PERMISSION_GRANTED;
        boolean writePermission = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_CONTACTS) == PackageManager.PERMISSION_GRANTED;

        if (!readPermission || !writePermission) {
            ActivityCompat.requestPermissions(
                    this,
                    new String[]{Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS},
                    PERMISSION_REQUEST_CODE
            );
        } else {
            fetchAndLogContacts();
        }
    }

    private void fetchAndLogContacts() {
        btnViewContacts.setVisibility(View.GONE);
        tvStatus.setVisibility(View.VISIBLE);

        // Ensure sample contacts are inserted if not already present
        insertSampleContactsIfNotExist();

        Uri uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;
        String[] projection = new String[]{
                ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME,
                ContactsContract.CommonDataKinds.Phone.NUMBER
        };

        try (Cursor cursor = getContentResolver().query(uri, projection, null, null, null)) {
            if (cursor != null && cursor.moveToFirst()) {
                logCursorContacts(cursor);
            } else {
                Log.d(TAG, "No contacts found.");
            }
        } catch (Exception e) {
            Log.e(TAG, "Error fetching contacts: " + e.getMessage(), e);
        }
    }

    private void logCursorContacts(Cursor cursor) {
        int nameIndex = cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME);
        int numberIndex = cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER);

        do {
            String name = (nameIndex != -1) ? cursor.getString(nameIndex) : "Unknown";
            String number = (numberIndex != -1) ? cursor.getString(numberIndex) : "No Number";
            Log.d(TAG, "Contact Name: " + name + " | Phone Number: " + number);
        } while (cursor.moveToNext());
    }

    private void insertSampleContactsIfNotExist() {
        String[][] sampleContacts = {
                {"karthik", "+91 9876543210"},
                {"ram", "+91 9876543211"},
                {"santhosh", "+91 9876543212"},
                {"elamukilan", "+91 9876543213"},
                {"manoj", "+91 9876543214"}
        };

        for (String[] contact : sampleContacts) {
            String name = contact[0];
            String phone = contact[1];

            if (!contactExists(name)) {
                ArrayList<ContentProviderOperation> ops = new ArrayList<>();
                ops.add(ContentProviderOperation.newInsert(ContactsContract.RawContacts.CONTENT_URI)
                        .withValue(ContactsContract.RawContacts.ACCOUNT_TYPE, null)
                        .withValue(ContactsContract.RawContacts.ACCOUNT_NAME, null)
                        .build());

                ops.add(ContentProviderOperation.newInsert(ContactsContract.Data.CONTENT_URI)
                        .withValueBackReference(ContactsContract.Data.RAW_CONTACT_ID, 0)
                        .withValue(ContactsContract.Data.MIMETYPE, ContactsContract.CommonDataKinds.StructuredName.CONTENT_ITEM_TYPE)
                        .withValue(ContactsContract.CommonDataKinds.StructuredName.DISPLAY_NAME, name)
                        .build());

                ops.add(ContentProviderOperation.newInsert(ContactsContract.Data.CONTENT_URI)
                        .withValueBackReference(ContactsContract.Data.RAW_CONTACT_ID, 0)
                        .withValue(ContactsContract.Data.MIMETYPE, ContactsContract.CommonDataKinds.Phone.CONTENT_ITEM_TYPE)
                        .withValue(ContactsContract.CommonDataKinds.Phone.NUMBER, phone)
                        .withValue(ContactsContract.CommonDataKinds.Phone.TYPE, ContactsContract.CommonDataKinds.Phone.TYPE_MOBILE)
                        .build());

                try {
                    getContentResolver().applyBatch(ContactsContract.AUTHORITY, ops);
                } catch (Exception e) {
                    Log.e(TAG, "Error inserting sample contact: " + e.getMessage());
                }
            }
        }
    }

    private boolean contactExists(String name) {
        Uri uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;
        String selection = ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " = ?";
        String[] selectionArgs = new String[]{name};

        try (Cursor cursor = getContentResolver().query(uri, null, selection, selectionArgs, null)) {
            return cursor != null && cursor.getCount() > 0;
        } catch (Exception e) {
            return false;
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == PERMISSION_REQUEST_CODE) {
            boolean allGranted = true;
            for (int result : grantResults) {
                if (result != PackageManager.PERMISSION_GRANTED) {
                    allGranted = false;
                    break;
                }
            }
            if (allGranted) {
                fetchAndLogContacts();
            } else {
                Toast.makeText(this, "Permission denied to read contacts.", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```

## OUTPUT
<img width="1920" height="1080" alt="Screenshot 2026-11-24 101346" src="https://github.com/user-attachments/assets/7635ebe0-1a3b-4661-b5d4-f313c062c6c2" />

<img width="1920" height="1080" alt="Screenshot 2026-11-24 101407" src="https://github.com/user-attachments/assets/d969d067-d761-4b97-89ae-3acfd1655de0" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0bb8bd69-24ba-47ce-a193-4676f3245755" />

## RESULT
Thus a Simple Android Application create your own content providers to get contacts details using Android Studio is developed and executed successfully.
