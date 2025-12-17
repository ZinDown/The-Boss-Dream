package com.buurep.joegyi.activities;

import android.content.Intent;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Bundle;
import android.os.CountDownTimer;
import android.os.Handler;
import android.widget.ProgressBar;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import org.json.JSONObject;

import java.util.TimeZone;

public class NewSpalsh extends AppCompatActivity {

    // ===== Fields (smali fields mapping) =====
    String A;
    String B;          // API မှလာတဲ့ string
    String C;
    String D;          // TimeZone ID (device)
    boolean E = false; // Activity decision flag

    ProgressBar x;
    CountDownTimer w;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_new_spalsh);

        x = findViewById(R.id.progressBar);

        // 🔹 TimeZone ကို D ထဲယူထားတာ (အဓိက point)
        D = TimeZone.getDefault().getID();
        // example: "Asia/Yangon", "Asia/Tokyo"

        // 🔹 Internet check
        if (!isConnected()) {
            showNoInternetDialog();
            return;
        }

        // 🔹 API Call (network request)
        callApi();
    }

    // ================= API CALL =================
    private void callApi() {
        /*
         * Smali:
         * R0.a.g0(this).a(new T0/g(url, success, error))
         *
         * အောက်မှာ simulation လုပ်ထားတယ်
         */

        new Handler().postDelayed(() -> {
            try {
                // ===== Simulated API JSON =====
                JSONObject json = new JSONObject();
                json.put("status", true);

                // B = allowed timezones / keywords
                json.put("B", "Asia/Yangon,Asia/Tokyo,Asia/Singapore");

                // success callback
                onApiSuccess(json);

            } catch (Exception e) {
                onApiFail();
            }
        }, 800);
    }

    // ================= API SUCCESS =================
    private void onApiSuccess(JSONObject json) {
        try {
            boolean status = json.getBoolean("status");

            if (status) {
                B = json.getString("B");

                // 🔹 B ကို split
                String[] items = B.split(",");

                // 🔹 D (TimeZone) နဲ့ compare
                for (String item : items) {
                    if (D.contains(item.trim())) {
                        E = true;   // ⭐ FLAG SET
                        break;
                    }
                }

                startTimer();

            } else {
                startTimer();
            }

        } catch (Exception e) {
            startTimer();
        }
    }

    private void onApiFail() {
        startTimer();
    }

    // ================= TIMER =================
    private void startTimer() {
        w = new CountDownTimer(1500, 500) {
            @Override
            public void onTick(long millisUntilFinished) { }

            @Override
            public void onFinish() {
                goNext();
            }
        }.start();
    }

    // ================= NEXT ACTIVITY DECISION =================
    private void goNext() {

        /*
         * Decompiled logic:
         *
         * if (E != 0)
         *   Main_Asone_Activity
         * else
         *   Main_Activity
         */

        if (E) {
            startActivity(new Intent(this,
                    com.buurep.joegyi.asone.Main_Asone_Activity.class));
        } else {
            startActivity(new Intent(this,
                    com.buurep.joegyi.sport.Main_Activity.class));
        }

        finish();
    }

    // ================= INTERNET =================
    private boolean isConnected() {
        ConnectivityManager cm =
                (ConnectivityManager) getSystemService(CONNECTIVITY_SERVICE);
        NetworkInfo net = cm.getActiveNetworkInfo();
        return net != null && net.isConnected();
    }

    // ================= DIALOG =================
    private void showNoInternetDialog() {
        new AlertDialog.Builder(this)
                .setTitle("No Internet")
                .setMessage("Please check your connection")
                .setCancelable(false)
                .setPositiveButton("OK", (d, w) -> finish())
                .show();
    }
}
