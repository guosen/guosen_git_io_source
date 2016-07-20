title: Android解析Html标签
date: 2016-07-20 15:09:05
tags:
- Android
- TextView
- Html
---
#富文本
## 在安卓显示富文本是一个比较常见的功能，我们对一个TextView的文本显示不同的样式，包括字体颜色、字体大小等等。
#实现方式
## 有两种简单的方式实现富文本的现实，有个共同点就是和html有关。一个是采用webView直接显示；一个是个用TextView解析标签。
#关于Html.formHtml()
##安卓自带的解析是有限的
```bash
<a href="...">
<b>
<big>
<blockquote>
<br>
<cite>
<dfn>
<div align="...">
<em>
<font size="..." color="..." face="...">
<h1>
<h2>
<h3>
<h4>
<h5>
<h6>
<i>
<img src="...">
<p>
<small>
<strike>
<strong>
<sub>
<sup>
<tt>
<u>
```
也就是说除了以上标签，其他的自定义标签就得自己实现；
#如何实现
## 自己实现Html.TagHandler
```bash
/**
 * HTML标签解析
 * 实现Html.TagHandle解析
 *
 * @author shouwang
 * @date 2016-7-20
 */
public class HtmlParser implements Html.TagHandler, ContentHandler {

    private final TagHandler     handler;
    private       ContentHandler wrapped;
    private       Editable       text;
    private ArrayDeque<Boolean> tagStatus = new ArrayDeque<Boolean>();

    public HtmlParser(TagHandler handler) {
        this.handler = handler;
    }

    public static Spanned buildSpannedText(String html, TagHandler handler) {
        return Html.fromHtml(html, null, new HtmlParser(handler));
    }

    public static String getValue(Attributes attributes, String name) {
        for (int i = 0, n = attributes.getLength(); i < n; i++) {
            if (name.equals(attributes.getLocalName(i)))
                return attributes.getValue(i);
        }
        return null;
    }

    @Override
    public void handleTag(boolean opening, String tag, Editable output, XMLReader xmlReader) {
        if (wrapped == null) {
            text = output;
            wrapped = xmlReader.getContentHandler();
            xmlReader.setContentHandler(this);
            tagStatus.addLast(Boolean.FALSE);
        }
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        boolean isHandled = handler.handleTag(true, localName, text, attributes);
        tagStatus.addLast(isHandled);
        if (!isHandled)
            wrapped.startElement(uri, localName, qName, attributes);
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        if (!tagStatus.removeLast())
            wrapped.endElement(uri, localName, qName);
        handler.handleTag(false, localName, text, null);
    }

    @Override
    public void setDocumentLocator(Locator locator) {
        wrapped.setDocumentLocator(locator);
    }

    @Override
    public void startDocument() throws SAXException {
        wrapped.startDocument();
    }

    @Override
    public void endDocument() throws SAXException {
        wrapped.endDocument();
    }

    @Override
    public void startPrefixMapping(String prefix, String uri) throws SAXException {
        wrapped.startPrefixMapping(prefix, uri);
    }

    @Override
    public void endPrefixMapping(String prefix) throws SAXException {
        wrapped.endPrefixMapping(prefix);
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        wrapped.characters(ch, start, length);
    }

    @Override
    public void ignorableWhitespace(char[] ch, int start, int length) throws SAXException {
        wrapped.ignorableWhitespace(ch, start, length);
    }

    @Override
    public void processingInstruction(String target, String data) throws SAXException {
        wrapped.processingInstruction(target, data);
    }

    @Override
    public void skippedEntity(String name) throws SAXException {
        wrapped.skippedEntity(name);
    }

    public interface TagHandler {
        boolean handleTag(boolean opening, String tag, Editable output, Attributes attributes);
    }
}

```

#对标签进行解析
##
```bash
public class HtmlTagFormatter {
    private static final String TAG_HANDLE_SPAN      = "span";
    private static final String TAG_HANDLE_STYLE     = "style";
    private static final String TAG_HANDLE_ALIGN     = "align";
    private static final String TAG_FONT_SIZE        = "font-size";
    private static final String TAG_BACKGROUND_COLOR = "background-color";
    private static final String Tag_FONT_COLOR       = "color";
    private static final String TAG_TEXT_ALIGN       = "text-align";
    private int startIndex;
    private int stopIndex;
    private String         styleContent   = "";
    private Vector<String> mListParents   = new Vector<String>();//用来标记列表(有序和无序列表)
    private int            mListItemCount = 0;//用来标记列表(有序和无序列表)

    public Spanned handlerHtmlContent(final Context context, String htmlContent) throws NumberFormatException {
        return HtmlParser.buildSpannedText(htmlContent, new HtmlParser.TagHandler() {
            @Override
            public boolean handleTag(boolean opening, String tag, Editable output, Attributes attributes) {
                if (tag.equals(TAG_HANDLE_SPAN)) {
                    //<style>标签的处理方式
                    if (opening) {
                        startIndex = output.length();
                        styleContent = HtmlParser.getValue(attributes, TAG_HANDLE_STYLE);
                    } else {
                        stopIndex = output.length();
                        if (!TextUtils.isEmpty(styleContent)) {
                            String[] styleValues = styleContent.split(";");
                            for (String styleValue : styleValues) {
                                String[] tmpValues = styleValue.split(":");
                                if (tmpValues != null && tmpValues.length > 0) { //(font-size=14px)
                                    if (TAG_FONT_SIZE.equals(tmpValues[0])) { //处理文字效果字体大小
                                        int size = Integer.valueOf(getAllNumbers(tmpValues[1]));
                                        output.setSpan(new AbsoluteSizeSpan(sp2px(context, size)), startIndex, stopIndex, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
                                    } else if (TAG_BACKGROUND_COLOR.equals(tmpValues[0])) { //处理背景效果
                                        output.setSpan(new BackgroundColorSpan(Color.parseColor(tmpValues[1])), startIndex, stopIndex, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
                                    } else if (Tag_FONT_COLOR.equals(tmpValues[0])) {//处理字体颜色<span style="color:"#000000">
                                        output.setSpan(new ForegroundColorSpan(Color.parseColor(tmpValues[1])), startIndex, stopIndex, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
                                    } else if(TAG_TEXT_ALIGN.equals(tmpValues[0])){
                                        handleAlignTag(output,tmpValues[1]);
                                    }
                                }
                            }
                        }
                    }
                } else if (tag.equals(TAG_HANDLE_ALIGN)) {
                    if (opening) {
                        startIndex = output.length();
                    } else {
                        stopIndex = output.length();
                    }
                }
                //列表标签的解析渲染
                if (tag.equals("ul") || tag.equals("ol") || tag.equals("dd")) {
                    if (opening) {
                        mListParents.add(tag);
                    } else mListParents.remove(tag);

                    mListItemCount = 0;
                } else if (tag.equals("li") && !opening) {
                    handleListTag(output);
                }
                return false;
            }
        });
    }

    //正则获取字体
    private static String getAllNumbers(String body) {
        Pattern pattern = Pattern.compile("\\d+");
        Matcher matcher = pattern.matcher(body);
        while (matcher.find()) {
            return matcher.group(0);
        }
        return "";
    }

    //处理列表标签
    private void handleListTag(Editable output) {
        if (mListParents.lastElement().equals("ul")) {
            output.append("\n");
            String[] split = output.toString().split("\n");

            int lastIndex = split.length - 1;
            int start = output.length() - split[lastIndex].length() - 1;
            output.setSpan(new BulletSpan(15), start, output.length(), 0);
        } else if (mListParents.lastElement().equals("ol")) {
            mListItemCount++;
            output.append("\n");
            String[] split = output.toString().split("\n");

            int lastIndex = split.length - 1;
            int start = output.length() - split[lastIndex].length() - 1;
            output.insert(start, mListItemCount + ". ");
            output.setSpan(new LeadingMarginSpan.Standard(15 * mListParents.size()), start, output.length(), 0);
        }
    }
    //处理<text-align>
    private void handleAlignTag(Editable output,String alignTag){
        AlignmentSpan.Standard as=new AlignmentSpan.Standard(Layout.Alignment.ALIGN_NORMAL);
        if(alignTag.equals("center")){
            as = new AlignmentSpan.Standard(
                    Layout.Alignment.ALIGN_CENTER);
        }else if(alignTag.equals("right")){
            as = new AlignmentSpan.Standard(
                    Layout.Alignment.ALIGN_OPPOSITE);
        }else if(alignTag.equals("left")){
            as = new AlignmentSpan.Standard(
                    Layout.Alignment.ALIGN_NORMAL);
        }
        // 参考:https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/SpannableStringBuilder.java
        // throw new RuntimeException("PARAGRAPH span must start at paragraph boundary");
        // AlignmentSpan继承ParagraphStyle;会检查前后是不是有换行符\n;没有的话抛出以上异常
        if(!"\n".equals(output.charAt(stopIndex-1))) {
            output.append("\n");
            output.setSpan(as, startIndex, stopIndex+1, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        }
    }

    public static int sp2px(Context context, float spValue) {
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValue * fontScale + 0.5f);
    }

```
#几个坑
##
1.各种预设的span类为我们带来了各种各样的样式，这些span分为两类，
  - 一类是CharacterStyle，也就是说是针对单个字符可以设置的样式
  - 另一类是ParagraphStyle，是针对段落进行的操作，ParagraphStyle的span运行的时候会检测目标段落前后是否有\n，如果没有的话会抛出错误"PARAGRAPH span must start at paragraph boundary"，这是一个大坑请务必要注意。
2.设置文字大小要进行单位转换
```bash
 final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValue * fontScale + 0.5f);
```
#参考
https://commonsware.com/blog/Android/2010/05/26/html-tags-supported-by-textview.html

https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/SpannableStringBuilder.java

http://lrdcq.com/me/read.php/37.htm