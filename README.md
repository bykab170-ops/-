import os

root = "CitizenDataApp"

# كل ملفات المشروع جاهزة بالكامل
structure = {
    "build.gradle": """// build.gradle (Project)
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:8.2.0'
        classpath 'com.google.gms:google-services:4.4.2'
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}""",
    "settings.gradle": "include ':app'\nrootProject.name = 'CitizenDataApp'\n",
    "app": {
        "build.gradle": """plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'com.google.gms.google-services'
}

android {
    namespace 'com.example.citizendataapp'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.citizendataapp"
        minSdk 21
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled false
        }
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib:1.9.10"
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.7.0'
    implementation 'com.google.android.material:material:1.12.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.3.3'
    implementation 'androidx.recyclerview:recyclerview:1.3.1'

    implementation platform('com.google.firebase:firebase-bom:33.1.2')
    implementation 'com.google.firebase:firebase-firestore-ktx'
}""",
        "src/main/AndroidManifest.xml": """<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.citizendataapp">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:label="CitizenDataApp"
        android:theme="@style/Theme.CitizenDataApp">

        <activity android:name=".CitizenListActivity" />
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>

</manifest>""",
        "src/main/java/com/example/citizendataapp/MainActivity.kt": """package com.example.citizendataapp

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import com.google.firebase.firestore.FirebaseFirestore

class MainActivity : AppCompatActivity() {

    private lateinit var db: FirebaseFirestore

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        db = FirebaseFirestore.getInstance()

        val fullName = findViewById<EditText>(R.id.fullName)
        val nationalId = findViewById<EditText>(R.id.nationalId)
        val phone = findViewById<EditText>(R.id.phone)
        val address = findViewById<EditText>(R.id.address)
        val job = findViewById<EditText>(R.id.job)
        val notes = findViewById<EditText>(R.id.notes)
        val saveButton = findViewById<Button>(R.id.saveButton)
        val showDataButton = findViewById<Button>(R.id.showDataButton)

        saveButton.setOnClickListener {
            val citizen = hashMapOf(
                "الاسم الكامل" to fullName.text.toString(),
                "الرقم الوطني" to nationalId.text.toString(),
                "رقم الهاتف" to phone.text.toString(),
                "العنوان" to address.text.toString(),
                "المهنة" to job.text.toString(),
                "الملاحظات" to notes.text.toString()
            )

            db.collection("citizens")
                .add(citizen)
                .addOnSuccessListener {
                    Toast.makeText(this, "تم حفظ البيانات بنجاح ✅", Toast.LENGTH_SHORT).show()
                    fullName.text.clear()
                    nationalId.text.clear()
                    phone.text.clear()
                    address.text.clear()
                    job.text.clear()
                    notes.text.clear()
                }
                .addOnFailureListener {
                    Toast.makeText(this, "حدث خطأ أثناء الحفظ ❌", Toast.LENGTH_SHORT).show()
                }
        }

        showDataButton.setOnClickListener {
            val intent = Intent(this, CitizenListActivity::class.java)
            startActivity(intent)
        }
    }
}""",
        "src/main/java/com/example/citizendataapp/CitizenListActivity.kt": """package com.example.citizendataapp

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.firebase.firestore.FirebaseFirestore

data class Citizen(
    val fullName: String = "",
    val n
