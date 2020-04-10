---
title: Search
permalink: /search/
date: 2020-04-09 06:45:44
---

<form action="https://duckduckgo.com/" method="get" class="search" id="eleventy-search"><div class="search-lo lo">
    <label for="search-term" class="question__label">Search Terms</label>
    <input type="search" name="q" id="question__field" value="site:www.aila.ngo" class="search-txt" autocomplete="off">
    <button type="submit" class="[ button ] [ font-base text-base weight-bold ]">Search</button>
</form>


<script src="/lunr/lunr.js"></script>


<div id="app">
	<input type="search" v-model="term"> <button @click="search">Search</button>
	<div v-if="results">
		<h3>Search Results</h3>
		<ul>
			<li v-for="result in results">
				<a :href="result.url"> {% raw %}{{ result.title }}{% endraw %}</a>
			</li>
		</ul>
		<p v-if="noResults">
		Sorry, no results were found.
		</p>
	</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
const app = new Vue({
	el:'#app',
	data:{
		docs:null,
		idx:null,
		term:'',
		results:null
	},
	async created() {
		let result = await fetch('/search.json');
		docs = await result.json();
		// assign an ID so it's easier to look up later, it will be the same as index
		this.idx = lunr(function () {
			this.ref('id');
			this.field('title');
			this.field('content');

			docs.forEach(function (doc, idx) {
				doc.id = idx;
				this.add(doc);
			}, this);
		});
		this.docs = docs;
	},
	computed: {
		noResults() {
			return this.results.length === 0;
		}
	},
	methods:{
		search() {
			console.log('search', this.term);
			let results = this.idx.search(this.term);

			// we need to add title, url from ref
			results.forEach(r => {
				r.title = this.docs[r.ref].title;
				r.url = this.docs[r.ref].url;
			});

			this.results = results;
		}
	}
});
</script>
