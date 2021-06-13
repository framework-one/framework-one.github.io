---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

# layout: home
---

<div class="blog-index">
	{% for post in site.posts %}
		{% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
		{% unless year == this_year %}
			{% assign year = this_year %}
			<h2>{{ year }}</h2>
			<hr/>
		{% endunless %}
		
		<article>
			{% include blog/year.html %}
		</article>
		
	{% endfor %}
</div>
