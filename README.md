/**بسم الله الرحمن الرحيم */
package com.vraces.vites.uddevallaprayertimes2;

import android.nfc.Tag;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import org.apache.poi.hssf.usermodel.HSSFDateUtil;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellValue;
import org.apache.poi.ss.usermodel.FormulaEvaluator;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";
    ArrayList<DataCall> uploadData;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();

        File file = new File(classLoader.getResource("res/raw/upt.xls").toString());

       final Button betn = findViewById(R.id.button);
        betn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

                ReadExcelData();


            }
        });
    }

    private void ReadExcelData ( ){



 /* Need to find out why arraylist is throwing hands**/

        try {
            InputStream inputStream = getResources().openRawResource(R.raw.upton);
            XSSFWorkbook workbook = new XSSFWorkbook(inputStream);
            XSSFSheet sheet = workbook.getSheetAt(0);
            int rowsCount = sheet.getPhysicalNumberOfRows();
            FormulaEvaluator formulaEvaluator = workbook.getCreationHelper().createFormulaEvaluator();
            StringBuilder sb = new StringBuilder();

            //outter loop, loops through rows
            for (int r = 1; r < rowsCount; r++) {
                Row row = sheet.getRow(r);
                int cellsCount = row.getPhysicalNumberOfCells();
                //inner loop, loops through columns
                for (int c = 0; c < cellsCount; c++) {
                    //handles if there are to many columns on the excel sheet.
                    if(c>6){
                        Log.e(TAG, "readExcelData: ERROR. Excel File Format is incorrect! " );
                        toastMessage("ERROR: Excel File Format is incorrect!");
                        break;
                    }else{
                        String value = getCellAsString(row, c, formulaEvaluator);
                        String cellInfo = "r:" + r + "; c:" + c + "; v:" + value;
                        Log.d(TAG, "readExcelData: Data from row: " + cellInfo);
                        sb.append(value+"\n");
                    }
                }
                sb.append(":");
            }
            Log.d(TAG, "readExcelData: STRINGBUILDER: " + sb.toString());

            try{
             parseStringBuilder(sb);
            }catch (IndexOutOfBoundsException e){
                Log.e(TAG, "IndexOutOfBounds - Before: "  + e.getMessage());
            }

        }catch (FileNotFoundException e) {
            Log.e(TAG, "readExcelData: FileNotFoundException. " + e.getMessage() );
        } catch (IOException e) {
            Log.e(TAG, "readExcelData: Error reading inputstream. " + e.getMessage() );
        } catch (IndexOutOfBoundsException e){
            Log.e(TAG, "IndexOutOfBounds: "  + e.getMessage());
        }
    }

    public void parseStringBuilder(StringBuilder mStringBuilder){
        Log.d(TAG, "parseStringBuilder: Started parsing.");

        // splits the sb into rows.
        String[] rows = mStringBuilder.toString().split(":");

        //Add to the ArrayList<XYValue> row by row
        for(int i=0; i<2555; i++) {
            //Split the columns of the rows
            Integer inter = rows.length;
            String parseInteger = inter.toString();

            String[] columns = rows[i].split(",");

            //use try catch to make sure there are no "" that try to parse into doubles.
            try{
                String datum = (columns[0]);
                String fajr = (columns[1]);
                String shuruk = (columns[2]);
                String dhuhr = (columns[3]);
                String asr = (columns[4]);
                String maghreb = (columns[5]);
                String ischa = (columns[6]);

                //String cellInfo = "(datum,fajr, shuruk, dhuhr, asr, maghreb , ischa): (" + datum + "," + fajr + ") (" + shuruk + "," + dhuhr + ")  (" + asr + "," + maghreb + ") (" + ischa + ")  "   ;
                Log.d(TAG, "ParseStringBuilder: Data from row: " + "cellInfo");

                //add the the uploadData ArrayList
                uploadData.add(new DataCall(datum, fajr, shuruk, dhuhr, asr, maghreb, ischa));

            }catch (NumberFormatException e){

                Log.e(TAG,"parseStringBuilder: NumberFormatException: " + e.getMessage());

            }catch (NullPointerException e) {
                Log.e(TAG,"Vad händer jau");
            }catch (IndexOutOfBoundsException e){
                Log.e(TAG, "IndexOutOfBounds Within Method: "  + e.getMessage());
            }

        }

       // printDataToLog();
    }
    private void printDataToLog() {
        Log.d(TAG, "printDataToLog: Printing data to log...");

        for(int i = 0; i< uploadData.size(); i++){
            String datum = uploadData.get(i).getDatum();
            String fajr = uploadData.get(i).getFajr();
            String shuruk = uploadData.get(i).getShuruk();
            String dhuhr = uploadData.get(i).getDhuhr();
            String asr = uploadData.get(i).getAsr();
            String maghreb = uploadData.get(i).getMaghreb();
            String ischa = uploadData.get(i).getIscha();

            Log.d(TAG, "\"(datum,fajr, shuruk, dhuhr, asr, maghreb , ischa): (\" + datum + \",\" + fajr + \") (\" + shuruk + \",\" + dhuhr + \")  (\" + asr + \",\" + maghreb + \") (\" + ischa + \")  \"   ");
        }
    }
    private String getCellAsString(Row row, int c, FormulaEvaluator formulaEvaluator) {
        String value = "";
        try {
            Cell cell = row.getCell(c);
            CellValue cellValue = formulaEvaluator.evaluate(cell);
            switch (cellValue.getCellType()) {
                case Cell.CELL_TYPE_BOOLEAN:
                    value = ""+cellValue.getBooleanValue();
                    break;
                case Cell.CELL_TYPE_NUMERIC:
                    double numericValue = cellValue.getNumberValue();
                    if(HSSFDateUtil.isCellDateFormatted(cell)) {
                        double date = cellValue.getNumberValue();
                        SimpleDateFormat formatter =
                                new SimpleDateFormat("yy/MM/dd");
                        value = formatter.format(HSSFDateUtil.getJavaDate(date));
                    } else {
                        value = ""+numericValue;
                    }
                    break;
                case Cell.CELL_TYPE_STRING:
                    value = ""+cellValue.getStringValue();
                    break;
                default:
            }
        } catch (NullPointerException e) {

            Log.e(TAG, "getCellAsString: NullPointerException: " + e.getMessage() );
        }
        return value;
    }
    private void toastMessage(String message){
        Toast.makeText(this,message, Toast.LENGTH_SHORT).show();
    }
}
