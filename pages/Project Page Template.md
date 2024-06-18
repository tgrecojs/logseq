- Project page
  template:: project page
  template-including-parent:: false
	- tags:: project page
	  icon:: 📂
	- [[Merkle Airdrop]]
	-
	- ### Project Meta
		- DOING #project <% current page %>
		- query-table:: false
		  #+BEGIN_QUERY
		  {:title [:h4 "Tasks related to <% current page %>"]
		  :query [:find (pull ?b [*])
		     :in $ ?current-page
		     :where
		     [?p :block/name ?current-page]
		     [?b :block/marker ?marker]
		  [?p :block/alias ?al]
		  (or [?b :block/refs ?p] [?b :block/refs ?al])
		  (or
		     [(= "NOW" ?marker)]
		     [(= "DOING" ?marker)]
		     [(= "WAITING" ?marker)]
		     [(= "LATER" ?marker)]
		  )
		  (not [?b :block/page ?p])
		  ]
		  :inputs [:current-page]
		  :result-transform (fn [result]
		                      (sort-by (fn [b]
		                                 (get b :block/priority "Z")) result))
		  :breadcrumb-show? false
		  :table-view? false
		  }
		  #+END_QUERY
		- query-table:: false
		  #+BEGIN_QUERY
		  {:title [:h4 "Checklist"]
		  :query (and (todo todo) (page <% current page %>))
		  :result-transform (fn [result]
		                      (sort-by (fn [b]
		                                 (get b :block/priority "Z")) result))
		  :breadcrumb-show? false
		  :table-view? false
		  }
		  #+END_QUERY
	- ### Notes
		- Project page
		  template:: project page
		  template-including-parent:: false
			- tags:: project page
			  icon:: 📂
			-
			- ### Project Meta
				- DOING #project <% current page %>
				- query-table:: false
				  #+BEGIN_QUERY
				  {:title [:h4 "Tasks related to <% current page %>"]
				  :query [:find (pull ?b [*])
				     :in $ ?current-page
				     :where
				     [?p :block/name ?current-page]
				     [?b :block/marker ?marker]
				  [?p :block/alias ?al]
				  (or [?b :block/refs ?p] [?b :block/refs ?al])
				  (or
				     [(= "NOW" ?marker)]
				     [(= "DOING" ?marker)]
				     [(= "WAITING" ?marker)]
				     [(= "LATER" ?marker)]
				  )
				  (not [?b :block/page ?p])
				  ]
				  :inputs [:current-page]
				  :result-transform (fn [result]
				                      (sort-by (fn [b]
				                                 (get b :block/priority "Z")) result))
				  :breadcrumb-show? false
				  :table-view? false
				  }
				  #+END_QUERY
				- query-table:: false
				  #+BEGIN_QUERY
				  {:title [:h4 "Checklist"]
				  :query (and (todo todo) (page <% current page %>))
				  :result-transform (fn [result]
				                      (sort-by (fn [b]
				                                 (get b :block/priority "Z")) result))
				  :breadcrumb-show? false
				  :table-view? false
				  }
				  #+END_QUERY
			- ### Notes
				-