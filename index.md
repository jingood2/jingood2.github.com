---
layout: page
title: jingood2's Knowledge Base
tagline: Individual Learning Space
---
{% include JB/setup %}

<div class="row">
    <div class="span10">
    {% include JB/recent_list %}
    </div>
    
    <div class="span2">
        <hr />
        <h2>Categories</h2>
        
        {% for category in site.categories %}
          <h3 id="{{ category[0] }}-ref">{{ category[0] | join: "/" }}</h3>
          <ul>
          {% assign pages_list = category[1] %}
          {% include JB/pages_list %}
          </ul>
        {% endfor %}
        
    </div> 
</div>

