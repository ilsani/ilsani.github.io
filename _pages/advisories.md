---
permalink: /advisories/
---

<p>This is a summary of the security bulletins that I published over the years.</p>

<table id="cust__hacking">
  <thead>
    <tr>
      <th>Date</th>
      <th>Title</th>
      <th>CVE</th>
    </tr>
  </thead>
  <tbody>
    {% for advisory in site.data.advisories %}
    <tr>
      <td>{{ advisory.date }}</td>
      <td><a href=
	     {% if advisory.link contains "http" and advisory.link contains "://" %}
	     "{{ advisory.link }}"
	     {% else %}
	     "{{ advisory.link | prepend: "/advisories/" | prepend: base_path }}"
	     {% endif %}
	     >{{ advisory.title }}</a></td>
      <td>{{ advisory.cvs | join: ", " }}</td>
    </tr>
    {% endfor %}
  </tbody>
  </table>
