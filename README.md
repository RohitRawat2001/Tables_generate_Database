# Tables_generate_Database
Table generate of database with forgein key


# dependency

```
			<dependency>
				<groupId>org.postgresql</groupId>
				<artifactId>postgresql</artifactId>
				<version>42.3.1</version>
			</dependency>
			<dependency>
				<groupId>org.apache.pdfbox</groupId>
				<artifactId>pdfbox</artifactId>
				<version>2.0.24</version>
			</dependency>
		<dependency>
			<groupId>com.itextpdf</groupId>
			<artifactId>kernel</artifactId>
			<version>7.1.16</version>
		</dependency>
		<dependency>
			<groupId>com.itextpdf</groupId>
			<artifactId>layout</artifactId>
			<version>7.1.16</version>
		</dependency>
```

# java code 

```
package com.student.Student;

import com.itextpdf.kernel.colors.ColorConstants;
import com.itextpdf.kernel.geom.PageSize;
import com.itextpdf.kernel.pdf.*;
import com.itextpdf.layout.Document;
import com.itextpdf.layout.element.*;
import com.itextpdf.layout.property.TextAlignment;
import com.itextpdf.layout.property.UnitValue;
import com.itextpdf.layout.property.VerticalAlignment;

import java.sql.*;

public class PostgresToPDFiText {
    public static void main(String[] args) {
        // PostgreSQL Connection details
        String url = "jdbc:postgresql://localhost:5432/rooms";
        String user = "postgres";
        String password = "rohit@001";

        // Output PDF file
        String pdfPath = "ERD_Report_A2_Landscape.pdf";

        // SQL Query to fetch ERD details
        String query = """
                SELECT 
                    c.table_name AS "Table",
                    c.column_name AS "Column",
                    c.data_type AS "Data Type",
                    c.is_nullable AS "Nullable",
                    tc.constraint_type AS "Constraint Type",
                    kcu.constraint_name AS "Constraint Name",
                    kcu.ordinal_position AS "Key Position",
                    ccu.table_name AS "Referenced Table",
                    ccu.column_name AS "Referenced Column"
                FROM information_schema.columns AS c
                LEFT JOIN information_schema.key_column_usage AS kcu 
                    ON c.table_name = kcu.table_name 
                    AND c.column_name = kcu.column_name
                LEFT JOIN information_schema.table_constraints AS tc 
                    ON kcu.constraint_name = tc.constraint_name
                LEFT JOIN information_schema.constraint_column_usage AS ccu 
                    ON tc.constraint_name = ccu.constraint_name
                WHERE c.table_schema = 'public'
                ORDER BY c.table_name, kcu.ordinal_position;
                """;

        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(query);
             PdfWriter writer = new PdfWriter(pdfPath);
             PdfDocument pdfDoc = new PdfDocument(writer);
             Document document = new Document(pdfDoc, PageSize.A2.rotate())) { // A2 page size in landscape mode

            // Set document margins
            document.setMargins(30, 30, 30, 30);

            // Title
            document.add(new Paragraph("PostgreSQL ERD Report (A2 Landscape Mode)")
                    .setBold().setFontSize(18).setFontColor(ColorConstants.BLUE)
                    .setTextAlignment(TextAlignment.CENTER));

            // Define table structure with adjusted column widths
            float[] columnWidths = {2f, 3f, 2f, 1.5f, 2.5f, 3f, 1.5f, 3f, 3f};
            Table table = new Table(UnitValue.createPercentArray(columnWidths));
            table.setWidth(UnitValue.createPercentValue(100)); // Full width

            // Define headers
            String[] headers = {"Table", "Column", "Data Type", "Nullable", "Constraint Type",
                    "Constraint Name", "Key Position", "Referenced Table", "Referenced Column"};

            // Add header row with styling
            for (String header : headers) {
                table.addHeaderCell(new Cell().add(new Paragraph(header)
                                .setBold().setFontSize(14))
                        .setBackgroundColor(ColorConstants.LIGHT_GRAY)
                        .setTextAlignment(TextAlignment.CENTER));
            }

            // Add data rows
            while (rs.next()) {
                for (String header : headers) {
                    String value = rs.getString(header);
                    Cell cell = new Cell().add(new Paragraph(value != null ? value : "N/A")
                            .setFontSize(12) // Adjusted font size for better readability
                            .setTextAlignment(TextAlignment.LEFT)
                            .setVerticalAlignment(VerticalAlignment.MIDDLE)
                            .setPadding(5)); // Adjust padding
                    cell.setKeepTogether(true); // Prevents text from splitting across pages
                    table.addCell(cell);
                }
            }

            // Add table to document
            document.add(table);
            document.close();

            System.out.println("✅ PDF generated successfully in A2 landscape mode: " + pdfPath);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

