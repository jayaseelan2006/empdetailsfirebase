
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
/*
Program to print the DatabaseTable using the firebasedatabase”.
Developed by:Jayaseelan U
Registeration Number :212223220039
*/
```
## ADD COUNTRYACTIVITY:
```
package com.example.crud;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class AddCountryActivity extends AppCompatActivity {

    private EditText etName, etCurrency;
    private DBManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_country);

        etName = findViewById(R.id.etName);
        etCurrency = findViewById(R.id.etCurrency);
        dbManager = new DBManager(this);
    }

    public void saveCountry(View view) {
        String name = etName.getText().toString();
        String currency = etCurrency.getText().toString();

        if (!name.isEmpty() && !currency.isEmpty()) {
            dbManager.addCountry(name, currency);
            Toast.makeText(this, "Country added", Toast.LENGTH_SHORT).show();
            finish();
        } else {
            Toast.makeText(this, "Please enter all fields", Toast.LENGTH_SHORT).show();
        }
    }
}

```
## COUNTRY:
```
package com.example.crud;

public class Country {
    private int id;
    private String name;
    private String currency;

    public Country(int id, String name, String currency) {
        this.id = id;
        this.name = name;
        this.currency = currency;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCurrency() { return currency; }

    @Override
    public String toString() {
        return name + " - " + currency;
    }
}

```
## COUNTRYLIST ACTIVITY:
```
package com.example.crud;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import java.util.List;

public class CountryListActivity extends AppCompatActivity {

    private ListView listView;
    private DBManager dbManager;
    private List<Country> countryList;
    private ArrayAdapter<Country> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_country_list);

        listView = findViewById(R.id.listView);
        dbManager = new DBManager(this);

        loadCountries();

        listView.setOnItemClickListener((parent, view, position, id) -> {
            Country selectedCountry = countryList.get(position);
            Intent intent = new Intent(CountryListActivity.this, ModifyCountryActivity.class);
            intent.putExtra("countryId", selectedCountry.getId());
            startActivity(intent);
        });
    }

    private void loadCountries() {
        countryList = dbManager.getAllCountries();
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, countryList);
        listView.setAdapter(adapter);
    }

    public void addCountry(View view) {
        startActivity(new Intent(this, AddCountryActivity.class));
    }

    @Override
    protected void onResume() {
        super.onResume();
        loadCountries(); // Refresh list when coming back
    }
}

```
## DATABASE HELPER:
```
package com.example.crud;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "CountryDB";
    private static final int DATABASE_VERSION = 1;

    public static final String TABLE_COUNTRIES = "countries";
    public static final String COLUMN_ID = "id";
    public static final String COLUMN_NAME = "name";
    public static final String COLUMN_CURRENCY = "currency";

    private static final String TABLE_CREATE =
            "CREATE TABLE " + TABLE_COUNTRIES + " (" +
                    COLUMN_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    COLUMN_NAME + " TEXT, " +
                    COLUMN_CURRENCY + " TEXT);";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(TABLE_CREATE);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_COUNTRIES);
        onCreate(db);
    }
}

```
## DBMANAGER:
```
package com.example.crud;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

import java.util.ArrayList;
import java.util.List;

public class DBManager {

    private DatabaseHelper dbHelper;
    private SQLiteDatabase database;

    public DBManager(Context context) {
        dbHelper = new DatabaseHelper(context);
        database = dbHelper.getWritableDatabase();
    }

    // Insert new country
    public long addCountry(String name, String currency) {
        ContentValues values = new ContentValues();
        values.put(DatabaseHelper.COLUMN_NAME, name);
        values.put(DatabaseHelper.COLUMN_CURRENCY, currency);
        return database.insert(DatabaseHelper.TABLE_COUNTRIES, null, values);
    }

    // Get all countries
    public List<Country> getAllCountries() {
        List<Country> countryList = new ArrayList<>();
        Cursor cursor = database.query(DatabaseHelper.TABLE_COUNTRIES, null, null, null, null, null, null);
        if (cursor.moveToFirst()) {
            do {
                int id = cursor.getInt(cursor.getColumnIndexOrThrow(DatabaseHelper.COLUMN_ID));
                String name = cursor.getString(cursor.getColumnIndexOrThrow(DatabaseHelper.COLUMN_NAME));
                String currency = cursor.getString(cursor.getColumnIndexOrThrow(DatabaseHelper.COLUMN_CURRENCY));
                countryList.add(new Country(id, name, currency));
            } while (cursor.moveToNext());
        }
        cursor.close();
        return countryList;
    }

    // Update a country
    public int updateCountry(int id, String name, String currency) {
        ContentValues values = new ContentValues();
        values.put(DatabaseHelper.COLUMN_NAME, name);
        values.put(DatabaseHelper.COLUMN_CURRENCY, currency);
        return database.update(DatabaseHelper.TABLE_COUNTRIES, values, DatabaseHelper.COLUMN_ID + "=?", new String[]{String.valueOf(id)});
    }

    // Delete a country
    public int deleteCountry(int id) {
        return database.delete(DatabaseHelper.TABLE_COUNTRIES, DatabaseHelper.COLUMN_ID + "=?", new String[]{String.valueOf(id)});
    }
}

```
MAIN ACTIVITY:
```
package com.example.crud;

import android.os.Bundle;

import com.google.android.material.bottomnavigation.BottomNavigationView;

import androidx.appcompat.app.AppCompatActivity;
import androidx.navigation.NavController;
import androidx.navigation.Navigation;
import androidx.navigation.ui.AppBarConfiguration;
import androidx.navigation.ui.NavigationUI;

import com.example.crud.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        BottomNavigationView navView = findViewById(R.id.nav_view);
        // Passing each menu ID as a set of Ids because each
        // menu should be considered as top level destinations.
        AppBarConfiguration appBarConfiguration = new AppBarConfiguration.Builder(
                R.id.navigation_home, R.id.navigation_dashboard, R.id.navigation_notifications)
                .build();
        NavController navController = Navigation.findNavController(this, R.id.nav_host_fragment_activity_main);
        NavigationUI.setupActionBarWithNavController(this, navController, appBarConfiguration);
        NavigationUI.setupWithNavController(binding.navView, navController);
    }

}
```
MODIFYCOUNTRY ACTIVITY:
```
package com.example.crud;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class ModifyCountryActivity extends AppCompatActivity {

    private EditText etName, etCurrency;
    private DBManager dbManager;
    private int countryId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_modify_country);

        etName = findViewById(R.id.etName);
        etCurrency = findViewById(R.id.etCurrency);
        dbManager = new DBManager(this);

        countryId = getIntent().getIntExtra("countryId", -1);
        Country country = dbManager.getAllCountries().stream()
                .filter(c -> c.getId() == countryId)
                .findFirst().orElse(null);

        if (country != null) {
            etName.setText(country.getName());
            etCurrency.setText(country.getCurrency());
        }
    }

    public void updateCountry(View view) {
        String name = etName.getText().toString();
        String currency = etCurrency.getText().toString();

        if (!name.isEmpty() && !currency.isEmpty()) {
            dbManager.updateCountry(countryId, name, currency);
            Toast.makeText(this, "Country updated", Toast.LENGTH_SHORT).show();
            finish();
        } else {
            Toast.makeText(this, "Please enter all fields", Toast.LENGTH_SHORT).show();
        }
    }

    public void deleteCountry(View view) {
        dbManager.deleteCountry(countryId);
        Toast.makeText(this, "Country deleted", Toast.LENGTH_SHORT).show();
        finish();
    }
}


```






## OUTPUT:
## COUNTRY LIST:

<img width="346" height="329" alt="Screenshot 2025-09-01 114215" src="https://github.com/user-attachments/assets/aa04c2c1-a8c6-48c2-94ff-97716c7da8bd" />
## ADD COUNTRY SCREEN:

<img width="344" height="297" alt="Screenshot 2025-09-01 114340" src="https://github.com/user-attachments/assets/9bf8b8e4-fc14-4e82-ac8c-03d24276a168" />
## MODIFY/DELETE COUNTRY SCREEN:

<img width="333" height="252" alt="Screenshot 2025-09-01 114434" src="https://github.com/user-attachments/assets/b5935536-1724-4b73-9c71-a1a6fd5fc1df" />


## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
