# Generate PDF & Excel in Spring Boot — Copy-Paste Codebase

A single Spring Boot project that exposes two endpoints:
- `GET /api/export/pdf` → downloads a PDF
- `GET /api/export/excel` → downloads an Excel (.xlsx) file

---

## 1. Project Structure

```
src/
└── main/
    ├── java/com/example/export/
    │   ├── ExportApplication.java
    │   ├── controller/ExportController.java
    │   ├── service/PdfService.java
    │   └── service/ExcelService.java
    └── resources/
        └── application.properties
pom.xml
```

---

## 2. `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.4</version>
    <relativePath/>
  </parent>

  <groupId>com.example</groupId>
  <artifactId>export-demo</artifactId>
  <version>1.0.0</version>
  <name>export-demo</name>

  <properties>
    <java.version>17</java.version>
  </properties>

  <dependencies>
    <!-- Spring Web -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- PDF: iText 7 (free Community edition) -->
    <dependency>
      <groupId>com.itextpdf</groupId>
      <artifactId>itext7-core</artifactId>
      <version>7.2.5</version>
      <type>pom</type>
    </dependency>

    <!-- Excel: Apache POI -->
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>5.2.5</version>
    </dependency>

    <!-- Lombok (optional, remove if you don't use it) -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

---

## 3. `ExportApplication.java`

```java
package com.example.export;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ExportApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExportApplication.class, args);
    }
}
```

---

## 4. `ExportController.java`

```java
package com.example.export.controller;

import com.example.export.service.ExcelService;
import com.example.export.service.PdfService;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;

@RestController
@RequestMapping("/api/export")
@RequiredArgsConstructor
public class ExportController {

    private final PdfService pdfService;
    private final ExcelService excelService;

    /**
     * Download PDF
     * URL: GET http://localhost:8080/api/export/pdf
     */
    @GetMapping("/pdf")
    public void exportPdf(HttpServletResponse response) throws IOException {
        response.setContentType("application/pdf");
        response.setHeader("Content-Disposition", "attachment; filename=report.pdf");
        pdfService.generate(response.getOutputStream());
    }

    /**
     * Download Excel
     * URL: GET http://localhost:8080/api/export/excel
     */
    @GetMapping("/excel")
    public void exportExcel(HttpServletResponse response) throws IOException {
        response.setContentType(
            "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setHeader("Content-Disposition", "attachment; filename=report.xlsx");
        excelService.generate(response.getOutputStream());
    }
}
```

---

## 5. `PdfService.java`

Uses **iText 7** to create a styled PDF with a title, a paragraph, and a table.

```java
package com.example.export.service;

import com.itextpdf.io.font.constants.StandardFonts;
import com.itextpdf.kernel.colors.ColorConstants;
import com.itextpdf.kernel.font.PdfFont;
import com.itextpdf.kernel.font.PdfFontFactory;
import com.itextpdf.kernel.pdf.PdfDocument;
import com.itextpdf.kernel.pdf.PdfWriter;
import com.itextpdf.layout.Document;
import com.itextpdf.layout.element.Cell;
import com.itextpdf.layout.element.Paragraph;
import com.itextpdf.layout.element.Table;
import com.itextpdf.layout.properties.TextAlignment;
import com.itextpdf.layout.properties.UnitValue;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

@Service
public class PdfService {

    // ── Sample data (replace with your real data / DB call) ──────────────────
    private static final String[] HEADERS = {"ID", "Name", "Department", "Salary"};
    private static final List<String[]> ROWS = List.of(
        new String[]{"1", "Alice Johnson",  "Engineering", "$95,000"},
        new String[]{"2", "Bob Smith",      "Marketing",   "$72,000"},
        new String[]{"3", "Carol Williams", "Finance",     "$88,000"},
        new String[]{"4", "David Brown",    "HR",          "$65,000"}
    );

    public void generate(OutputStream out) throws IOException {
        PdfWriter writer   = new PdfWriter(out);
        PdfDocument pdf    = new PdfDocument(writer);
        Document document  = new Document(pdf);

        PdfFont boldFont   = PdfFontFactory.createFont(StandardFonts.HELVETICA_BOLD);
        PdfFont normalFont = PdfFontFactory.createFont(StandardFonts.HELVETICA);

        // ── Title ─────────────────────────────────────────────────────────────
        document.add(new Paragraph("Employee Report")
            .setFont(boldFont)
            .setFontSize(20)
            .setTextAlignment(TextAlignment.CENTER)
            .setMarginBottom(4));

        document.add(new Paragraph("Generated by Spring Boot Export Demo")
            .setFont(normalFont)
            .setFontSize(10)
            .setFontColor(ColorConstants.GRAY)
            .setTextAlignment(TextAlignment.CENTER)
            .setMarginBottom(20));

        // ── Table ─────────────────────────────────────────────────────────────
        Table table = new Table(UnitValue.createPercentArray(new float[]{10, 30, 25, 20}))
            .useAllAvailableWidth();

        // Header row
        for (String header : HEADERS) {
            table.addHeaderCell(
                new Cell()
                    .add(new Paragraph(header).setFont(boldFont).setFontSize(11))
                    .setBackgroundColor(ColorConstants.DARK_GRAY)
                    .setFontColor(ColorConstants.WHITE)
                    .setPadding(6)
            );
        }

        // Data rows — alternating background
        boolean alternate = false;
        for (String[] row : ROWS) {
            for (String value : row) {
                Cell cell = new Cell()
                    .add(new Paragraph(value).setFont(normalFont).setFontSize(10))
                    .setPadding(5);
                if (alternate) {
                    cell.setBackgroundColor(ColorConstants.LIGHT_GRAY);
                }
                table.addCell(cell);
            }
            alternate = !alternate;
        }

        document.add(table);
        document.close();
    }
}
```

---

## 6. `ExcelService.java`

Uses **Apache POI** to create a styled `.xlsx` workbook with header formatting and auto-sized columns.

```java
package com.example.export.service;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

@Service
public class ExcelService {

    // ── Sample data (replace with your real data / DB call) ──────────────────
    private static final String[] HEADERS = {"ID", "Name", "Department", "Salary"};
    private static final List<Object[]> ROWS = List.of(
        new Object[]{1, "Alice Johnson",  "Engineering", 95000},
        new Object[]{2, "Bob Smith",      "Marketing",   72000},
        new Object[]{3, "Carol Williams", "Finance",     88000},
        new Object[]{4, "David Brown",    "HR",          65000}
    );

    public void generate(OutputStream out) throws IOException {
        try (Workbook workbook = new XSSFWorkbook()) {
            Sheet sheet = workbook.createSheet("Employees");

            // ── Header style ─────────────────────────────────────────────────
            CellStyle headerStyle = workbook.createCellStyle();
            Font headerFont = workbook.createFont();
            headerFont.setBold(true);
            headerFont.setColor(IndexedColors.WHITE.getIndex());
            headerStyle.setFont(headerFont);
            headerStyle.setFillForegroundColor(IndexedColors.DARK_BLUE.getIndex());
            headerStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
            headerStyle.setAlignment(HorizontalAlignment.CENTER);
            headerStyle.setBorderBottom(BorderStyle.THIN);

            // ── Header row ───────────────────────────────────────────────────
            Row headerRow = sheet.createRow(0);
            for (int i = 0; i < HEADERS.length; i++) {
                Cell cell = headerRow.createCell(i);
                cell.setCellValue(HEADERS[i]);
                cell.setCellStyle(headerStyle);
            }

            // ── Alternating row styles ────────────────────────────────────────
            CellStyle evenStyle = buildRowStyle(workbook, IndexedColors.LIGHT_TURQUOISE);
            CellStyle oddStyle  = buildRowStyle(workbook, IndexedColors.WHITE);

            // ── Currency format for Salary column ────────────────────────────
            DataFormat format = workbook.createDataFormat();
            CellStyle currencyEven = workbook.createCellStyle();
            currencyEven.cloneStyleFrom(evenStyle);
            currencyEven.setDataFormat(format.getFormat("$#,##0"));

            CellStyle currencyOdd = workbook.createCellStyle();
            currencyOdd.cloneStyleFrom(oddStyle);
            currencyOdd.setDataFormat(format.getFormat("$#,##0"));

            // ── Data rows ────────────────────────────────────────────────────
            int rowIndex = 1;
            for (Object[] rowData : ROWS) {
                Row row = sheet.createRow(rowIndex);
                boolean isEven = rowIndex % 2 == 0;

                for (int col = 0; col < rowData.length; col++) {
                    Cell cell = row.createCell(col);
                    Object value = rowData[col];

                    if (value instanceof Integer || value instanceof Long) {
                        cell.setCellValue(((Number) value).doubleValue());
                        // Last column is salary → use currency style
                        cell.setCellStyle(col == rowData.length - 1
                            ? (isEven ? currencyEven : currencyOdd)
                            : (isEven ? evenStyle : oddStyle));
                    } else {
                        cell.setCellValue(value.toString());
                        cell.setCellStyle(isEven ? evenStyle : oddStyle);
                    }
                }
                rowIndex++;
            }

            // ── Auto-size all columns ─────────────────────────────────────────
            for (int i = 0; i < HEADERS.length; i++) {
                sheet.autoSizeColumn(i);
                // Add a small buffer so text isn't clipped
                sheet.setColumnWidth(i, sheet.getColumnWidth(i) + 512);
            }

            workbook.write(out);
        }
    }

    private CellStyle buildRowStyle(Workbook workbook, IndexedColors bgColor) {
        CellStyle style = workbook.createCellStyle();
        style.setFillForegroundColor(bgColor.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        style.setBorderBottom(BorderStyle.THIN);
        style.setBorderLeft(BorderStyle.THIN);
        style.setBorderRight(BorderStyle.THIN);
        style.setBottomBorderColor(IndexedColors.GREY_25_PERCENT.getIndex());
        return style;
    }
}
```

---

## 7. `application.properties`

```properties
spring.application.name=export-demo
server.port=8080
```

---

## 8. Run & Test

```bash
# Build and run
mvn spring-boot:run

# Download PDF (browser or curl)
curl -O http://localhost:8080/api/export/pdf

# Download Excel
curl -O http://localhost:8080/api/export/excel
```

---

## 9. Swap in Real Data (DB / JPA)

Replace the static `ROWS` lists in both services with a repository call:

```java
// In PdfService or ExcelService, inject your repository:
@Autowired
private EmployeeRepository employeeRepository;

// Then inside generate():
List<Employee> employees = employeeRepository.findAll();
for (Employee emp : employees) {
    // build your row from emp fields
}
```

---

## 10. Quick Reference

| Feature | Library | Maven artifact |
|---|---|---|
| PDF generation | iText 7 | `com.itextpdf:itext7-core:7.2.5` |
| Excel (.xlsx) | Apache POI | `org.apache.poi:poi-ooxml:5.2.5` |
| Web layer | Spring MVC | `spring-boot-starter-web` |

> **Tip:** For very large Excel files (100k+ rows), switch from `XSSFWorkbook` to `SXSSFWorkbook` — it streams rows to disk and avoids `OutOfMemoryError`.

```java
// Streaming workbook for large datasets
Workbook workbook = new SXSSFWorkbook(100); // keep 100 rows in memory
```
