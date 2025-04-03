# Import-csv-file-database
Import-csv-file-database


CSV file upload in Java where the header values are dynamic and can contain commas (subrate scenario), 
we need a structured approach:

**Key Considerations:**
1. Handle Comma within Values: Use a CSV parser that supports escaping or quotes, e.g., OpenCSV or Apache Commons CSV.
2. Dynamic Fields Growth: Use a flexible data structure such as Map<String, String> or List<String> for headers.
3. Upload Handling: Spring Boot + MultipartFile if you're handling file uploads via REST API.

**1. Maven Dependencies (Using Apache Commons CSV)**
```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.9.0</version>
</dependency>
```
**2. CSV File Upload Controller (Spring Boot)**

```
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.commons.csv.CSVRecord;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;

@RestController
@RequestMapping("/api/csv")
public class CSVUploadController {

    @PostMapping("/upload")
    public Map<String, List<String>> uploadCSV(@RequestParam("file") MultipartFile file) {
        Map<String, List<String>> csvData = new LinkedHashMap<>();
        
        try (Reader reader = new InputStreamReader(file.getInputStream(), StandardCharsets.UTF_8);
             CSVParser csvParser = new CSVParser(reader, CSVFormat.DEFAULT.withFirstRecordAsHeader().withTrim().withQuote('"'))) {

            // Dynamically fetch headers
            List<String> headers = new ArrayList<>(csvParser.getHeaderMap().keySet());
            headers.forEach(header -> csvData.put(header, new ArrayList<>()));

            // Read records
            for (CSVRecord record : csvParser) {
                for (String header : headers) {
                    csvData.get(header).add(record.get(header));
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
        return csvData;
    }
}
```

**3. Sample CSV Upload Request (Postman / Frontend)**
```
POST /api/csv/upload
Content-Type: multipart/form-data
file: [Sample_csv_file.csv]
```
**4. Sample CSV File**
```
"ID", "Name", "Description", "Price"
1, "Item A", "High quality, durable", 100
2, "Item B", "Low price, best value", 50

➡ Handles Comma within Values Properly using " as quotes.
```
**5. Expected Output (JSON Response)**
```
{
    "ID": ["1", "2"],
    "Name": ["Item A", "Item B"],
    "Description": ["High quality, durable", "Low price, best value"],
    "Price": ["100", "50"]
}
```
