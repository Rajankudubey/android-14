MainActivity.kt (Main Activity with UI):

import android.content.Context
import android.content.Intent
import android.content.SharedPreferences
import android.os.Bundle
import android.telephony.TelephonyManager
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.Switch
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private lateinit var statusMessageEditText: EditText
    private lateinit var activateSwitch: Switch
    private lateinit var saveButton: Button

    private lateinit var sharedPreferences: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        statusMessageEditText = findViewById(R.id.statusMessageEditText)
        activateSwitch = findViewById(R.id.activateSwitch)
        saveButton = findViewById(R.id.saveButton)

        sharedPreferences = getSharedPreferences("MyPrefs", Context.MODE_PRIVATE)

        saveButton.setOnClickListener {
            saveStatusMessage()
        }

        activateSwitch.setOnCheckedChangeListener { _, isChecked ->
            saveActivationState(isChecked)
        }

        // Initialize the UI based on the saved activation state
        activateSwitch.isChecked = sharedPreferences.getBoolean("is_active", false)
        statusMessageEditText.setText(sharedPreferences.getString("status_message", ""))
    }

    private fun saveStatusMessage() {
        val statusMessage = statusMessageEditText.text.toString()
        val editor = sharedPreferences.edit()
        editor.putString("status_message", statusMessage)
        editor.apply()
        Toast.makeText(this, "Status message saved", Toast.LENGTH_SHORT).show()
    }

    private fun saveActivationState(isActive: Boolean) {
        val editor = sharedPreferences.edit()
        editor.putBoolean("is_active", isActive)
        editor.apply()
    }
}
CallReceiver.kt (BroadcastReceiver for phone state):
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.telephony.TelephonyManager
import android.widget.Toast

class CallReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        if (intent?.action == "android.intent.action.PHONE_STATE") {
            val state = intent.getStringExtra(TelephonyManager.EXTRA_STATE)
            if (state == TelephonyManager.EXTRA_STATE_RINGING) {
                val sharedPreferences = context?.getSharedPreferences("MyPrefs", Context.MODE_PRIVATE)
                val isActive = sharedPreferences?.getBoolean("is_active", false) ?: false
                if (isActive) {
                    val statusMessage = sharedPreferences?.getString("status_message", "")
                    Toast.makeText(context, "Status: $statusMessage", Toast.LENGTH_LONG).show()
                }
            }
        }
    }
}

activity_main.xml (Layout for the main activity):

<EditText
    android:id="@+id/statusMessageEditText"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="Enter Status Message"/>

<Switch
    android:id="@+id/activateSwitch"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="16dp"
    android:text="Activate"/>

<Button
    android:id="@+id/saveButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    android:layout_marginTop="16dp"
    android:text="Save"/>

