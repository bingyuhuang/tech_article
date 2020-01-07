---
title: Java常用工具类
tags: Java, 工具类
renderNumberedHeading: true
grammar_cjkRuby: true
---
[toc]
### 字符编码判断和转换
```
import java.io.UnsupportedEncodingException;
 
/**
 * 判断字符编码
 *
 */
public class CharacterCodingUtil {
 
    /**
     * 判断是否为ISO-8859-1
     *
     * @return
     */
    public static boolean checkISO(String str) {
        boolean flag = java.nio.charset.Charset.forName("ISO-8859-1").newEncoder().canEncode(str);
        return flag;
    }
 
    /**
     * 判断是否为UTF-8
     *
     * @return
     */
    public static boolean checkUTF(String str) {
 
        boolean flag = java.nio.charset.Charset.forName("UTF-8").newEncoder().canEncode(str);
        return flag;
    }
 
    public static boolean checkUnicode(String str) {
 
        boolean flag = java.nio.charset.Charset.forName("unicode").newEncoder().canEncode(str);
        return flag;
    }
 
    /**
     * Title: getEncoding
     * Description: 判断字符编码
     *
     * @param str
     * @return
     */
    public static String getEncoding(String str) {
        String encode = "unicode";
        try {
            if (str.equals(new String(str.getBytes(encode), encode))) {
                String s = encode;
                return s;
            }
        } catch (Exception exception) {
        }
        encode = "ISO-8859-1";
        try {
            if (str.equals(new String(str.getBytes(encode), encode))) {
                String s1 = encode;
                return s1;
            }
        } catch (Exception exception1) {
        }
        encode = "UTF-8";
        try {
            if (str.equals(new String(str.getBytes(encode), encode))) {
                String s2 = encode;
                return s2;
            }
        } catch (Exception exception2) {
        }
        encode = "GBK";
        try {
            if (str.equals(new String(str.getBytes(encode), encode))) {
                String s3 = encode;
                return s3;
            }
        } catch (Exception exception3) {
        }
        return "";
    }
 
    /**
     * Title: isoToutf8
     * Description: ISO-8859-1 编码 转 UTF-8
     *
     * @param str
     * @return
     */
    public static String isoToutf8(String str) {
        try {
            if (!checkISO(str))
                return str;
            return new String(str.getBytes("ISO-8859-1"), "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return str;
        }
    }
 
    /**
     * Title: utf8Toiso
     * Description: UTF-8 编码 转 ISO-8859-1
     *
     * @param str
     * @return
     */
    public static String utf8Toiso(String str) {
        try {
            if (!checkUTF(str))
                return str;
            return new String(str.getBytes("utf-8"), "iso-8859-1");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return str;
        }
    }
 
    /**
     *
     * @param unicode
     * @return
     */
    public static String unicodeToCn(String unicode) {
        /** 以 \ u 分割，因为java注释也能识别unicode，因此中间加了一个空格 */
        String[] strs = unicode.split("\\\\u");
        String returnStr = "";
        // 由于unicode字符串以 \ u 开头，因此分割出的第一个字符是""。
        for (int i = 1; i < strs.length; i++) {
            returnStr += (char) Integer.valueOf(strs[i], 16).intValue();
        }
        return returnStr;
    }
 
    /**
     *
     * @param cn
     * @return
     */
    public static String cnToUnicode(String cn) {
        char[] chars = cn.toCharArray();
        String returnStr = "";
        for (int i = 0; i < chars.length; i++) {
            returnStr += "\\u" + Integer.toString(chars[i], 16);
        }
        return returnStr;
    }
 
 
}
```
### 提取汉字首字符
使用[依赖包](https://mvnrepository.com/artifact/com.belerweb/pinyin4j/2.5.0)
```
import net.sourceforge.pinyin4j.PinyinHelper;
import net.sourceforge.pinyin4j.format.HanyuPinyinCaseType;
import net.sourceforge.pinyin4j.format.HanyuPinyinOutputFormat;
import net.sourceforge.pinyin4j.format.HanyuPinyinToneType;
import net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination;
 
/**
 * 提取汉字首字母工具类
 * <p>Title: ChineseToFirstLetterUtil</p>
 * <p>Description: </p>
 * <p>Company: www.itcast.cn</p> 
 * @version 1.0
 */
public class ChineseToFirstLetterUtil {
 	public static String ChineseToFirstLetter(String c) {
		String string = "";
		char b;
		int a = c.length();
		for (int k = 0; k < a; k++) {
			b = c.charAt(k);
			String d = String.valueOf(b);
			String str = converterToFirstSpell(d);
			String s = str.toUpperCase();
			String g = s;
			char h;
			int j = g.length();
			for (int y = 0; y <= 0; y++) {
				h = g.charAt(0);
				string += h;
			}
		}
		return string;
	}
 
	public static String converterToFirstSpell(String chines) {
		String pinyinName = "";
		char[] nameChar = chines.toCharArray();
		HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
		defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		for (int i = 0; i < nameChar.length; i++) {
			String s = String.valueOf(nameChar[i]);
			if (s.matches("[\\u4e00-\\u9fa5]")) {
				try {
					String[] mPinyinArray = PinyinHelper.toHanyuPinyinStringArray(nameChar[i], defaultFormat);
					pinyinName += mPinyinArray[0];
				} catch (BadHanyuPinyinOutputFormatCombination e) {
					e.printStackTrace();
				}
			} else {
				pinyinName += nameChar[i];
			}
		}
		return pinyinName;
	}
	
	public static void main(String[] args) {
		System.err.println(ChineseToFirstLetter("犯我中华者虽远必诛"));
		 
	}
}
```