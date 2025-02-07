# Tables_generate_Database or json
Table generate of database with forgein key

Like This Output :



![Screenshot 2025-02-07 152926](https://github.com/user-attachments/assets/986881ec-4ad6-445b-811c-d2c68b946302)

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

```java
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
        String password = "";

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

# Json format using python

```py
import json
import psycopg2



conn = psycopg2.connect(
    dbname="rooms",
    user="postgres",
    password="",
    host="localhost",
    port="5432"
)
cur = conn.cursor()


data_type_abbr = {
    "character varying": "VCHAR",
    "character": "CHAR",
    "text": "TEXT",
    "integer": "INT",
    "bigint": "BIGINT",
    "boolean": "BOOL",
    "date": "DATE",
    "timestamp without time zone": "TS",
    "timestamp with time zone": "TSTZ",
    "decimal": "DEC",
    "numeric": "DEC",
    "real": "REAL",
    "double precision": "DOUBLE",
    "smallint": "SMALLINT",
    "uuid": "UUID",
    "json": "JSON"
}

schema_name = 'public'
output_file = "table_schemas.json"


schema_data = {}


cur.execute(f"""
    SELECT table_name
    FROM information_schema.tables
    WHERE table_schema = %s AND table_type = 'BASE TABLE';
""", (schema_name,))


tables = [row[0] for row in cur.fetchall()]

for table_name in tables:
    
    cur.execute(f"""
        SELECT column_name, data_type, is_nullable, column_default
        FROM information_schema.columns
        WHERE table_name = %s;
    """, (table_name,))

    
    columns = cur.fetchall()

    
    cur.execute(f"""
        SELECT kcu.column_name
        FROM information_schema.table_constraints tco
        JOIN information_schema.key_column_usage kcu
        ON tco.constraint_name = kcu.constraint_name
        WHERE tco.table_name = %s AND tco.constraint_type = 'PRIMARY KEY';
    """, (table_name,))

    
    primary_key = set([col[0] for col in cur.fetchall()])

    
    cur.execute(f"""
        SELECT kcu.column_name, ccu.table_name AS foreign_table, ccu.column_name AS foreign_column
        FROM information_schema.key_column_usage kcu
        JOIN information_schema.constraint_column_usage ccu
        ON kcu.constraint_name = ccu.constraint_name
        WHERE kcu.table_name = %s AND kcu.constraint_name IN (
            SELECT constraint_name
            FROM information_schema.table_constraints
            WHERE table_name = %s AND constraint_type = 'FOREIGN KEY'
        );
    """, (table_name, table_name))

    
    foreign_keys = {fk[0]: fk[1:] for fk in cur.fetchall()}

    
    cur.execute(f"""
        SELECT kcu.column_name
        FROM information_schema.table_constraints tco
        JOIN information_schema.key_column_usage kcu
        ON tco.constraint_name = kcu.constraint_name
        WHERE tco.table_name = %s AND tco.constraint_type = 'UNIQUE';
    """, (table_name,))

    
    unique_constraints = set([col[0] for col in cur.fetchall()])

    
    table_schema = {
        "table": table_name,
        "columns": []
    }

    for column in columns:
        column_name, data_type, is_nullable, column_default = column
        abbr_data_type = data_type_abbr.get(data_type.lower(), data_type)
        nullable = "NOT NULL" if is_nullable == "NO" else ""
        constraints = []

        
        if column_name in primary_key:
            constraints.append("PK")

        
        if column_name in unique_constraints:
            constraints.append("UNIQUE")

        
        fk_info = ""
        if column_name in foreign_keys:
            fk_table, fk_column = foreign_keys[column_name]
            fk_info = f"FK->{fk_table}.{fk_column}"

        
        column_str = f"{column_name}({abbr_data_type},{nullable}{',DEFAULT' + column_default if column_default else ''}{',' + ','.join(constraints) if constraints else ''}{',' + fk_info if fk_info else ''})"

        
        table_schema["columns"].append(column_str)

    
    schema_data[table_name] = table_schema


with open(output_file, "w") as file:
    json.dump(schema_data, file, indent=4)


cur.close()
conn.close()

print(f"Schema details for all tables have been saved to {output_file}.")
```


# output Like this :
```
 "app_users": {
        "table": "app_users",
        "columns": [
            "id(VCHAR,NOT NULL,PK)",
            "create_date(TS,)",
            "last_update_date(TS,)",
            "apppassword(VCHAR,NOT NULL)",
            "authentication_type(VCHAR,NOT NULL)",
            "disable_flag(BOOL,NOT NULL)",
            "end_date(TS,)",
            "password_expiry_flag(BOOL,)",
            "userid(VCHAR,NOT NULL)",
            "create_by(VCHAR,,FK->app_users.id)",
            "org_idx(VCHAR,,FK->app_organisations.id)",
            "update_by(VCHAR,,FK->app_users.id)",
            "employee_idx(VCHAR,,FK->employees.id)",
            "last_login(TS,)",
            "deleted(BOOL,,DEFAULTfalse)",
            "app_version(VCHAR,)",
            "build_number(VCHAR,)",
            "login_device(VCHAR,)",
            "system_admin_flag(BOOL,,DEFAULTfalse)",
            "enable_device_virtualassistant(BOOL,,DEFAULTfalse)"
        ]
    },
    "spatial_ref_sys": {
        "table": "spatial_ref_sys",
        "columns": [
            "srid(INT,NOT NULL,PK)",
            "auth_name(VCHAR,)",
            "auth_srid(INT,)",
            "srtext(VCHAR,)",
            "proj4text(VCHAR,)"
        ]
    },
    "app_navigations_cloud": {
        "table": "app_navigations_cloud",
        "columns": [
            "id(VCHAR,NOT NULL,PK)",
            "create_date(TS,)",
            "last_update_date(TS,)",
            "access(JSON,)",
            "create_by(VCHAR,,FK->app_users.id)",
            "org_idx(VCHAR,,FK->app_organisations.id)",
            "update_by(VCHAR,,FK->app_users.id)",
            "deleted(BOOL,,DEFAULTfalse)"
        ]
    },
```
