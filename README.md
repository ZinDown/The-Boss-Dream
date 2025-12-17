<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:card="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#F2F2F2">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:id="@+id/txtLive2D"
            android:text="--"
            android:textSize="64sp"
            android:textStyle="bold"
            android:gravity="center"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

        <TextView
            android:id="@+id/txtUpdate"
            android:text="Updated --"
            android:gravity="center"
            android:textColor="#4CAF50"
            android:paddingBottom="16dp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

        <!-- ===== 4 CARDS ===== -->
        <android.support.v7.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            card:cardCornerRadius="20dp"
            card:cardElevation="6dp"
            card:cardBackgroundColor="#4C7CF3"
            android:layout_marginBottom="12dp">

            <LinearLayout
                android:padding="16dp"
                android:orientation="vertical"
                android:layout_width="match_parent"
                android:layout_height="wrap_content">

                <TextView
                    android:id="@+id/time1"
                    android:text="11:00 AM"
                    android:textSize="20sp"
                    android:textStyle="bold"
                    android:textColor="#FFF"
                    android:gravity="center"/>

                <View android:layout_width="match_parent"
                    android:layout_height="1dp"
                    android:background="#80FFFFFF"
                    android:layout_margin="12dp"/>

                <LinearLayout
                    android:weightSum="3"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content">

                    <TextView android:id="@+id/set1"
                        android:layout_weight="1"
                        android:textColor="#FFF"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"/>

                    <TextView android:id="@+id/value1"
                        android:layout_weight="1"
                        android:gravity="center"
                        android:textColor="#FFF"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"/>

                    <TextView android:id="@+id/d1"
                        android:layout_weight="1"
                        android:gravity="end"
                        android:textSize="26sp"
                        android:textStyle="bold"
                        android:textColor="#FFEB3B"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"/>
                </LinearLayout>
            </LinearLayout>
        </android.support.v7.widget.CardView>

        <!-- copy 3 times -->
        <include layout="@layout/activity_main"/>

    </LinearLayout>
</ScrollView>


package com.twod.live;

import android.os.Bundle;
import android.os.Handler;
import android.support.v7.app.AppCompatActivity;
import android.widget.TextView;

import org.json.JSONArray;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    TextView txtLive2D, txtUpdate;
    TextView[] set = new TextView[4];
    TextView[] value = new TextView[4];
    TextView[] d = new TextView[4];

    String API = "https://api.thaistock2d.com/live";
    Handler handler = new Handler();

    Runnable autoRefresh = new Runnable() {
        @Override
        public void run() {
            load();
            handler.postDelayed(this, 3000); // 3 seconds
        }
    };

    @Override
    protected void onCreate(Bundle b) {
        super.onCreate(b);
        setContentView(R.layout.activity_main);

        txtLive2D = findViewById(R.id.txtLive2D);
        txtUpdate = findViewById(R.id.txtUpdate);

        set[0] = findViewById(R.id.set1);
        value[0] = findViewById(R.id.value1);
        d[0] = findViewById(R.id.d1);

        set[1] = findViewById(R.id.set2);
        value[1] = findViewById(R.id.value2);
        d[1] = findViewById(R.id.d2);

        set[2] = findViewById(R.id.set3);
        value[2] = findViewById(R.id.value3);
        d[2] = findViewById(R.id.d3);

        set[3] = findViewById(R.id.set4);
        value[3] = findViewById(R.id.value4);
        d[3] = findViewById(R.id.d4);

        autoRefresh.run();
    }

    void load() {
        new Thread(() -> {
            try {
                URL url = new URL(API);
                HttpURLConnection c = (HttpURLConnection) url.openConnection();
                c.setRequestMethod("GET");

                BufferedReader br = new BufferedReader(
                        new InputStreamReader(c.getInputStream()));

                StringBuilder sb = new StringBuilder();
                String l;
                while ((l = br.readLine()) != null) sb.append(l);

                br.close();
                c.disconnect();

                JSONObject json = new JSONObject(sb.toString());
                runOnUiThread(() -> parse(json));

            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }

    void parse(JSONObject json) {
        try {
            txtUpdate.setText("Updated: " + json.getString("server_time"));
            txtLive2D.setText(json.getJSONObject("live").getString("twod"));

            JSONArray arr = json.getJSONArray("result");

            for (int i = 0; i < 4 && i < arr.length(); i++) {
                JSONObject o = arr.getJSONObject(i);
                set[i].setText("Set\n" + o.getString("set"));
                value[i].setText("Value\n" + o.getString("value"));
                d[i].setText(o.getString("twod"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}





