<%* if (tp.file.title.charAt(0) == "{") { %>
<%-tp.file.include("[[Book]]")%>
<%* } else if (tp.file.title.charAt(0) == "@") { %>
<%-tp.file.include("[[Person]]")%>
<%* } else if (tp.file.title.charAt(0) == "!") { %>
<%-tp.file.include("[[Learning-Journal]]")%>
<%* } else if (tp.file.title.charAt(0) == "%") { %>
<%-tp.file.include("[[Podcast]]")%>
<%* } else if (tp.file.title.charAt(0) == "+") { %>
<%-tp.file.include("[[Video]]")%>
<%* } else if (tp.file.title.charAt(0) == "(") { %>
<%-tp.file.include("[[Article]]")%>
<%* } else if (tp.file.title.charAt(0) == "&") { %>
<%-tp.file.include("[[Paper]]")%>
<%* } else if (tp.file.title.charAt(0) == "=") { %>
<%-tp.file.include("[[Thought]]")%>
<%* } else if (tp.file.title.charAt(0) == "}") { %>
<%-tp.file.include("[[Something-Can-Go-Here]]")%>
<%* } else { %>
<%-tp.file.include("[[New]]")%>
<%* } _%>