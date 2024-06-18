- Query tasks on this page Template
  template:: page tasks
  template-including-parent:: false
	- {{query (and (todo todo doing) (page <% current page %>))}}