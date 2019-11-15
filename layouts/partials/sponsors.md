
<p> {{.Site.Data.sponsors.msg}} </p>

<ul class='list-group'>
{{range .Site.Data.sponsors.list}}
<li class='list-group-item'> {{.name}} </li>
{{end}}
</ul>
<br>
<p> {{safeHTML .Site.Data.sponsors.msg2}} </p>



