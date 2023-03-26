title: Java日期转换工具类
date: 2023-03-04 10:04:40
categories:
    - Java
tags: 
    - CodeCheetSheet
    - Java
---


```java
package util.common;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.regex.Pattern;

/**
 * DateUtil
 * jdk8 or higher
 * @author sicmatr1x@outlook.com
 * @date 2023/3/4 20:22
 */
public class DateUtil {

    // Empty checks
    //-----------------------------------------------------------------------

    public static boolean isEmpty(final CharSequence cs) {
        return cs == null || cs.length() == 0;
    }

    // Date formats
    //-----------------------------------------------------------------------

    public static final String DEFAULT_DATETIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    private static final Map<String, String> FORMATS = new HashMap<>();
    private static final Map<String, String> FORMATS_REGEX = new HashMap<>();

    static {
        FORMATS.put("yyyyMMdd", "^\\d{4}\\d{1,2}\\d{1,2}$");
        FORMATS.put("yyyy-MM-dd", "^\\d{4}-\\d{1,2}-\\d{1,2}$");
        FORMATS.put("yyyy/MM/dd", "^\\d{4}/\\d{1,2}/\\d{1,2}$");
        FORMATS.put("yyyy.MM.dd", "^\\d{4}\\.\\d{1,2}\\.\\d{1,2}$");
        FORMATS.put("yyyy-MM-dd HH:mm:ss", "^(?:(?!0000)[0-9]{4}-(?:(?:0[1-9]|1[0-2])-(?:0[1-9]|1[0-9]|2[0-8])|(?:0[13-9]|1[0-2])-(?:29|30)|(?:0[13578]|1[02])-31)|(?:[0-9]{2}(?:0[48]|[2468][048]|[13579][26])|(?:0[48]|[2468][048]|[13579][26])00)-02-29) (20|21|22|23|[0-1][0-9]):[0-5][0-9]:[0-5][0-9]$");

        FORMATS_REGEX.put("^\\d{4}\\d{1,2}\\d{1,2}$", "yyyyMMdd");
        FORMATS_REGEX.put("^\\d{4}-\\d{1,2}-\\d{1,2}$", "yyyy-MM-dd");
        FORMATS_REGEX.put("^\\d{4}/\\d{1,2}/\\d{1,2}$", "yyyy/MM/dd");
        FORMATS_REGEX.put("^\\d{4}\\.\\d{1,2}\\.\\d{1,2}$", "yyyy.MM.dd");
        FORMATS_REGEX.put("^(?:(?!0000)[0-9]{4}-(?:(?:0[1-9]|1[0-2])-(?:0[1-9]|1[0-9]|2[0-8])|(?:0[13-9]|1[0-2])-(?:29|30)|(?:0[13578]|1[02])-31)|(?:[0-9]{2}(?:0[48]|[2468][048]|[13579][26])|(?:0[48]|[2468][048]|[13579][26])00)-02-29) (20|21|22|23|[0-1][0-9]):[0-5][0-9]:[0-5][0-9]$", "yyyy-MM-dd HH:mm:ss");
    }

    /**
     * test date string is match the format
     * @param format date format
     * @param dateStr date string
     * @return true: match; false: not match
     */
    public static boolean test(String format, String dateStr) {
        if (isEmpty(format) || isEmpty(dateStr)) {
            return false;
        }
        if (!FORMATS.containsKey(format)) {
            return false;
        }
        String formatRegex = FORMATS.get(format);
        return Pattern.matches(formatRegex, dateStr);
    }

    // Date formats: string -> java.util.Date
    //-----------------------------------------------------------------------

    /**
     * recognize the date string format
     * @param dateStr date string
     * @return date format
     */
    public static Optional<String> recognizeFormat(String dateStr) {
        if (isEmpty(dateStr)) {
            return Optional.empty();
        }
        return FORMATS_REGEX.entrySet().stream()
                .filter(e -> Pattern.matches(e.getKey(), dateStr))
                .map(Map.Entry::getValue)
                .findFirst();
    }

    /**
     * recognize the date string format, if not recognize than return default string
     * @param dateStr date string
     * @return date format
     */
    public static String recognizeFormat(String dateStr, String defaultStr) {
        return recognizeFormat(dateStr).orElse(defaultStr);
    }

    /**
     * smart recognize the date string format, and parse to date object
     *
     * @param dateStr date string need to parse
     * @return date object
     * @throws ParseException parse error
     */
    public static Date smartParse(String dateStr) throws ParseException {
        if (isEmpty(dateStr)) {
            return null;
        }
        Optional<String> format = recognizeFormat(dateStr);
        if (format.isPresent()) {
            SimpleDateFormat sdf = new SimpleDateFormat(format.get());
            return sdf.parse(dateStr);
        }
        return null;
    }

    /**
     * smart recognize the date string format, and parse to support date object
     *
     * @param dateStr date string need to parse
     * @param clazz support class: java.util.Date, java.time.LocalDate, java.time.LocalTime, java.time.LocalDateTime
     * @return support date object
     * @throws ParseException parse error
     */
    public static Object smartParse(String dateStr, Class clazz) throws ParseException {
        Date date = smartParse(dateStr);
        if (date == null || clazz == null) {
            return null;
        }
        if (clazz == Date.class) {
            return date;
        } else if (clazz == LocalDate.class) {
            return toLocalDate(date);
        } else if (clazz == LocalTime.class) {
            return toLocalTime(date);
        } else if (clazz == LocalDateTime.class) {
            return toLocalDateTime(date);
        }
        return null;
    }

    // Date formats: date object -> string
    //-----------------------------------------------------------------------

    /**
     * some support class date object convert to date string,
     * if not support or illegal dateObj than return an empty string
     *
     * @param dateObj support class: java.util.Date, java.time.LocalDate, java.time.LocalTime, java.time.LocalDateTime
     * @param formatStr date format
     * @return date string
     */
    public static String toString(Object dateObj, String formatStr) {
        if (dateObj == null) {
            return "";
        }
        if (dateObj instanceof String) {
            return (String) dateObj;
        }
        if (isEmpty(formatStr)) {
            formatStr = DEFAULT_DATETIME_FORMAT;
        }
        if (dateObj instanceof Date) {
            SimpleDateFormat sdf = new SimpleDateFormat(formatStr);
            return sdf.format(dateObj);
        }
        if (dateObj instanceof LocalDate) {
            DateTimeFormatter dtf = DateTimeFormatter.ofPattern(formatStr);
            return ((LocalDate) dateObj).format(dtf);
        } else if (dateObj instanceof LocalTime) {
            DateTimeFormatter dtf = DateTimeFormatter.ofPattern(formatStr);
            return ((LocalTime) dateObj).format(dtf);
        } else if (dateObj instanceof LocalDateTime) {
            DateTimeFormatter dtf = DateTimeFormatter.ofPattern(formatStr);
            return ((LocalDateTime) dateObj).format(dtf);
        }
        return "";
    }

    /**
     * java.time.LocalDate -> String
     *
     * @param localDate java.time.LocalDate
     * @param formatStr format
     * @return date string
     */
    public static String toString(LocalDate localDate, String formatStr) {
        if (localDate == null) {
            return "";
        }
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern(formatStr);
        return localDate.format(dtf);
    }

    /**
     * java.time.LocalTime -> String
     *
     * @param localTime java.time.LocalTime
     * @param formatStr format
     * @return date string
     */
    public static String toString(LocalTime localTime, String formatStr) {
        if (localTime == null) {
            return "";
        }
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern(formatStr);
        return localTime.format(dtf);
    }

    /**
     * java.time.LocalDateTime -> String
     *
     * @param localDateTime java.time.LocalDateTime
     * @param formatStr format
     * @return date string
     */
    public static String toString(LocalDateTime localDateTime, String formatStr) {
        if (localDateTime == null) {
            return "";
        }
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern(formatStr);
        return localDateTime.format(dtf);
    }

    // java.util.Date -> java.time.LocalDate
    //                -> java.time.LocalTime
    //                -> java.time.LocalDateTime
    //-----------------------------------------------------------------------

    /**
     * java.util.Date -> java.time.LocalDate
     *
     * @param date java.util.Date
     * @param zoneId A time-zone ID, if null than use system default
     * @return java.time.LocalDate
     */
    public static LocalDate toLocalDate(Date date, ZoneId zoneId) {
        Instant instant = date.toInstant();
        if (zoneId == null) {
            zoneId = ZoneId.systemDefault();
        }
        return instant.atZone(zoneId).toLocalDate();
    }

    /**
     * java.util.Date -> java.time.LocalDate
     *
     * @param date java.util.Date
     * @return java.time.LocalDate
     */
    public static LocalDate toLocalDate(Date date) {
        return toLocalDate(date, null);
    }

    /**
     * java.util.Date -> java.time.LocalTime
     *
     * @param date java.util.Date
     * @param zoneId A time-zone ID, if null than use system default
     * @return java.time.LocalTime
     */
    public static LocalTime toLocalTime(Date date, ZoneId zoneId) {
        Instant instant = date.toInstant();
        if (zoneId == null) {
            zoneId = ZoneId.systemDefault();
        }
        return instant.atZone(zoneId).toLocalTime();
    }

    /**
     * java.util.Date -> java.time.LocalTime
     *
     * @param date java.util.Date
     * @return java.time.LocalTime
     */
    public static LocalTime toLocalTime(Date date) {
        return toLocalTime(date, null);
    }

    /**
     * java.util.Date -> java.time.LocalDateTime
     *
     * @param date java.util.Date
     * @param zoneId A time-zone ID, if null than use system default
     * @return java.time.LocalDateTime
     */
    public static LocalDateTime toLocalDateTime(Date date, ZoneId zoneId) {
        Instant instant = date.toInstant();
        if (zoneId == null) {
            zoneId = ZoneId.systemDefault();
        }
        return instant.atZone(zoneId).toLocalDateTime();
    }

    /**
     * java.util.Date -> java.time.LocalTime
     *
     * @param date java.util.Date
     * @return java.time.LocalDateTime
     */
    public static LocalDateTime toLocalDateTime(Date date) {
        return toLocalDateTime(date, null);
    }

}
```


