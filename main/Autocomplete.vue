<template>
  <div class="autocomplete">
    <label :for="id">{{label}}</label>
    <textarea v-if="textarea" :id="id" :rows="rows" :cols="cols" class="autocomplete-input" :placeholder="placeholder" @focusout="focusout" @focus="focus" @keydown.13="chooseItem" @keydown.tab="chooseItem" @keydown.40="moveDown" @keydown.38="moveUp" v-model="inputValue"
      type="text"></textarea>
    <input v-else :id="id" class="autocomplete-input" :placeholder="placeholder" @focusout="focusout" @focus="focus" @keydown.13="chooseItem" @keydown.tab="chooseItem" @keydown.40="moveDown" @keydown.38="moveUp" v-model="inputValue" type="text">


    <ul :class="{
      'autocomplete-list': true,
      [id+'-list']: true
    }" v-if="searchMatch.length > 0">
      <li :class="{active: selectedIndex === index}" v-for="(result, index) in searchMatch" @click="selectItem(index), chooseItem()" v-html="highlightWord(result)">

      </li>
    </ul>
  </div>
  <!--
  E.g
   <autocomplete
    label="Custom items:"
    :items="['all', 'auto', 'complete', 'items']"
    placeholder="all, auto..." />
  -->
</template>
    <!-- Scripts  -->
  <script type="text/javascript" src="assets/js/v-autocomplete.js">
  const standardItems = [
  	"input",
  	"h1",
  	"h2",
  	"h3",
  	"h4",
  	"h5",
  	"h6",
  	"span",
  	"div",
  	"textarea",
  	"margin",
  	"padding",
  	"display",
  	"background",
  	"background-color",
  	"background-size",
  	"background-repeat",
  	"position",
  	"top",
  	"left",
  	"right",
  	"bottom"
  ];
  const Autocomplete = Vue.component("autocomplete", {
  	template: "#autocomplete-tpl",
  	props: ["items", "placeholder", "label", "textarea", "rows", "cols"],
  	data() {
  		return {
  			id: 'input-' + parseInt(Math.random() * 1000),
  			inputValue: "",
  			searchMatch: [],
  			selectedIndex: 0,
  			clickedChooseItem: false,
  			wordIndex: 0
  		};
  	},
  	mounted() {
      const _self = this;
      document.querySelector('#' + this.id)
        .addEventListener('input', function() {
          const caret = getCaretCoordinates(this, this.selectionEnd);

          if (_self.searchMatch.length > 0) {
            const element = document.querySelectorAll('.' + _self.id + '-list');

            if (element[0]) {
              element[0].style.top = caret.top + 40 + 'px';
              element[0].style.left = caret.left + 'px';
            }
          }
        });
    },
  	computed: {
  		listToSearch() {
  			if (typeof this.items !== "undefined" && this.items.length > 0) {
  				return this.items;
  			} else {
  				return standardItems;
  			}
  		},
  		currentWord() {
  			return this.inputValue.replace(/(\r\n|\n|\r)/gm, ' ').split(' ')[this.wordIndex];
  		},
  		inputSplitted() {
  			return this.inputValue.replace(/(\r\n|\n|\r)/gm, ' ').split(" ");
  		}
  	},
  	watch: {
  		inputValue() {
  			this.focus();
  			console.log(this.inputSplitted)
  			this.selectedIndex = 0;
  			this.wordIndex = this.inputSplitted.length - 1;
  		}
  	},
  	methods: {
  		highlightWord(word) {
  			const regex = new RegExp("(" + this.currentWord + ")", "g");
  			return word.replace(regex, '<mark>$1</mark>');
  		},
  		setWord(word) {
  			let currentWords = this.inputValue.replace(/(\r\n|\n|\r)/gm, '__br__ ').split(' ');
  			currentWords[this.wordIndex] = currentWords[this.wordIndex].replace(this.currentWord, word + ' ');
  			this.wordIndex += 1;
  			this.inputValue = currentWords.join(' ').replace(/__br__\s/g, '\n');
  		},
  		moveDown() {
  			if (this.selectedIndex < this.searchMatch.length - 1) {
  				this.selectedIndex++;
  			}
  		},
  		moveUp() {
  			if (this.selectedIndex !== -1) {
  				this.selectedIndex--;
  			}
  		},
  		selectItem(index) {
  			this.selectedIndex = index;
  			this.chooseItem();
  		},
  		chooseItem(e) {
  			this.clickedChooseItem = true;

  			if (this.selectedIndex !== -1 && this.searchMatch.length > 0) {
  				if (e) {
  					e.preventDefault();
  				}
  				this.setWord(this.searchMatch[this.selectedIndex]);
  				this.selectedIndex = -1;
  			}
  		},
  		focusout(e) {
  			setTimeout(() => {
  				if (!this.clickedChooseItem) {
  					this.searchMatch = [];
  					this.selectedIndex = -1;
  				}
  				this.clickedChooseItem = false;
  			}, 100);
  		},
  		focus() {
  			this.searchMatch = [];
  			if (this.currentWord !== "") {
  				this.searchMatch = this.listToSearch.filter(
  					el => el.indexOf(this.currentWord) >= 0
  				);
  			}
  			if (
  				this.searchMatch.length === 1 &&
  				this.currentWord === this.searchMatch[0]
  			) {
  				this.searchMatch = [];
  			}
  		}
  	}
  });

  new Vue({
  	el: "#app"
  });


  </script>
<style scopeds>
  .autocomplete {
    position: relative;
  }
  .autocomplete label {
    display: block;
    margin-bottom: 5px;
    font-size: 14px;
    font-weight: 100;
  }
  .autocomplete-input {
    padding: 7px 10px;
    width: 93%;
    border: 1px solid #ddd;
    border-radius: 4px;
    outline: none;
  }
  .autocomplete-input:focus {
    border-color: #000;
  }
  .autocomplete-list {
    position: absolute;
    z-index: 2;
    overflow: auto;
    min-width: 250px;
    max-height: 150px;
    margin: 0;
    margin-top: 5px;
    padding: 0;
    border: 1px solid #eee;
    list-style: none;
    border-radius: 4px;
    background-color: #fff;
    box-shadow: 0 5px 25px rgba(0, 0, 0, 0.05);
  }
  .autocomplete-list li {
    margin: 0;
    padding: 8px 15px;
    border-bottom: 1px solid #f5f5f5;
  }
  .autocomplete-list li:last-child {
    border-bottom: 0;
  }
  .autocomplete-list li:hover, .autocomplete-list li.active {
    background-color: #f5f5f5;
  }
</style>
