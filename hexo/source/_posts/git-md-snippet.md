title: git_md_snippet
date: 2015-10-23 22:53:25
tags: git
categories: tools
---

- Markdown_Link_(mdlink).sublime-snippet

```
<snippet>
 <content><![CDATA[
[${1:Display_Text}](${2:http://example.com/} ${3:"$2"})
]]></content>
 <tabTrigger>mdlink</tabTrigger>
 <scope>text.html.markdown.multimarkdown, text.html.markdown</scope>
 <description>Insert Link</description>
</snippet>
```

- Markdown_Image_(mdimg).sublime-snippet

```
<snippet>
 <content><![CDATA[
![${1:Some_Text}](${2:url_to_image} ${3:"$1"})
]]></content>
 <tabTrigger>mdimg</tabTrigger>
 <scope>text.html.markdown.multimarkdown, text.html.markdown</scope>
 <description>Insert Image</description>
</snippet>
```

- Markdown_Anchor_(mdarch).sublime-snippet

```
<snippet>
 <content><![CDATA[
[${1:Display_Text}][${2:id}]$5
[$2]:${3:http://example.com/} ${4:"$3"}
]]></content>
 <tabTrigger>mdacr</tabTrigger>
 <scope>text.html.markdown.multimarkdown, text.html.markdown</scope>
 <description>Link Anchor</description>
</snippet>
```

- Markdown_Footnote_(mdfn).sublime-snippet

```
<snippet>
 <content><![CDATA[
[^${1:Footnote}]$3
[^$1]:${2:Footnote_Text}
]]></content>
 <tabTrigger>mdfn</tabTrigger>
 <scope>text.html.markdown.multimarkdown, text.html.markdown</scope>
 <description>Insert Footnote</description>
</snippet>
```
