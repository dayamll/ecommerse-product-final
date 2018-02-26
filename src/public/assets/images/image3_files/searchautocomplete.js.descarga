/**
 * MageWorx
 *
 * NOTICE OF LICENSE
 *
 * This source file is subject to the MageWorx EULA that is bundled with
 * this package in the file LICENSE.txt.
 * It is also available through the world-wide-web at this URL:
 * http://www.mageworx.com/LICENSE-1.0.html
 *
 * DISCLAIMER
 *
 * Do not edit or add to this file if you wish to upgrade the extension
 * to newer versions in the future. If you wish to customize the extension
 * for your needs please refer to http://www.mageworx.com/ for more information
 *
 * @category   MageWorx
 * @package    MageWorx_SearchAutocomplete
 * @copyright  Copyright (c) 2013 MageWorx (http://www.mageworx.com/)
 * @license    http://www.mageworx.com/LICENSE-1.0.html
 */

/**
 * Search Autocomplete extension
 *
 * @category   MageWorx
 * @package    MageWorx_SearchAutocomplete
 * @author     MageWorx Dev Team 
 */
var SearchAutocomplete = Class.create({

    initialize: function(searchFieldId, resultContainerId, url, interval, message) {
        this.searchFieldId = searchFieldId;
        this.containerId = resultContainerId;
        this.interval = interval;
        this.timer = 0;
        this.currentQuery = '';
        this.cache = [];
        this.url = url;
        this.message = '';
        this.delay = true;
        this.searchField = $(searchFieldId);
        this.container = $(resultContainerId);
        this.searchField.observe('keyup', this.getInput.bind(this));
        this.searchField.observe('focus', this.onFocus.bind(this));
        this.searchField.observe('blur', this.onBlur.bind(this));
        this.searchParameter = '';
        this.isValid = false;
        this.oldPosition = 0;
        if(message != undefined){
            this.message = message;
        }
        Ajax.Request.prototype.abort = function() {
            this.transport.onreadystatechange = Prototype.emptyFunction;
            this.transport.abort();
            Ajax.activeRequestCount--;
        };
        if(typeof String.prototype.trim !== 'function') { // ie fix 
            String.prototype.trim = function() {
                return this.replace(/^\s+|\s+$/g, ''); 
            }
        }
        this.rb = null;
        this.addResetButton();
        if(this.searchField.getValue() != message){
            this.resetButton(true);
        }
        document.observe('click',this.documentClick);
    },
    onFocus: function(event){
        var value = this.searchField.getValue().trim();
        if(value == this.message){
            this.searchField.setValue('');
        } else if(value.length > 1 && typeof(this.cache[value]) == 'string') {
            this.showResult(value.toLowerCase());
        }
    },
    onBlur: function(event){
        var value = this.searchField.getValue().trim();
        if (value.length == 0) {
            this.searchField.setValue(this.message);
        }
    },
    getInput: function(event){
        var value = this.searchField.getValue().trim();
        if (this.validate(value)) {
            clearTimeout(this.timer);
            var queryCode = value.toLowerCase();
            if(this.searchParameter.length > 0){
                if (this.cache[queryCode + '_' + this.searchParameter] != undefined) {
                    this.showResult(queryCode);
                } else {
                    var object = this;
                    this.timer = setTimeout(function(){
                        if(!object.isValid) return;
                        object.search();
                    },this.delay?this.interval:0);
                }
            }
            else {
                if (typeof(this.cache[queryCode]) == 'string') {
                    this.showResult(queryCode);
                } else {
                    var object = this;
                    this.timer = setTimeout(function(){
                        if(!object.isValid) return;
                        object.search();
                    },this.interval);
                }
            }
        }
        else {
            this.stopRequest(false);
        }
    },
    search: function() {
        var value = this.currentQuery;
        var queryCode = value.toLowerCase();
        var object = this;
        this.stopRequest(true);
        var params = {};
        params['q'] = value;
        if(object.searchParameter.length != 0){
            this.delay = true;
            params['a'] = this.searchParameter;
        }
        this.request = new Ajax.Request(this.url,{
            method: 'GET',
            onSuccess: function(responce) {
                if(object.searchParameter.length != 0){
                    queryCode += '_' + object.searchParameter;
                }
                object.cache[queryCode] = responce.responseText;
                object.spinner(false);
                object.showResult(value.toLowerCase());
            },
            parameters: params
        });
        
    },
    validate: function(query) {
        var isValid = false;
        if(query.length > 1) {   
            this.resetButton(true);
            if (query != this.message) {
                var words = query.split(' ');
                for (var i=0,length = words.length; i<length; i++) {
                    if(words[i].length > 1) {
                        isValid = true;
                        this.currentQuery = query;
                        break;
                    }
                }
            }
        } else {
            this.container.hide();
            this.resetButton();
        }
        this.isValid = isValid;
        return isValid;
    },
    showResult: function(queryCode){  
        if(this.searchParameter.length != 0){
            queryCode += '_' + this.searchParameter;
        }
        this.stopRequest(false);
        this.container.update(this.cache[queryCode]);
        this.container.show();
        
        var c = $$('.searchautocomplete-search').first();
        if(c != undefined){
            var form = $$('.form-search').first();
            var button = $$('.form-search button[type="submit"]','.xsearch-submit').first();
            var offset = c.offsetWidth - (this.searchField.offsetLeft + this.searchField.offsetWidth);//form.offsetWidth - button.offsetLeft;
            c.setStyle({'left': -offset + 'px'});
            c.style.left = -offset + 'px';
        }
        
        
    },
    spinner: function(enabled){
        if(enabled == true){
            this.searchField.addClassName('spinner');
            this.oldPosition = this.searchField.style.backgroundPosition;
            var w = this.searchField.offsetWidth - 33;
            this.searchField.setStyle({'background-position-x': w + 'px'})
            this.searchField.style.backgroundPosition = w + 'px';
        }
        else {
            this.searchField.removeClassName('spinner');
            this.searchField.style.backgroundPosition = this.oldPosition;
        }
    },
    stopRequest: function(spinner){
        if(typeof this.request != 'undefined') this.request.abort();
        this.spinner(spinner);
    },
    addSearchParameter: function(parameter){
        this.searchParameter = parameter;
        this.delay = false;
        this.getInput();
    },
    documentClick: function(event){
        var element = event.toElement || event.target;
        if(element.id == 'search' || element.className == 'xsearch-change' || element.className == 'xsearch-dropdown') return;
        var el = element.up('.searchautocomplete-search');
        if(el != undefined && el.innerHTML.length > 1) return;
        el = element.up('.xsearch-select');
        if(el != undefined && el.innerHTML.length > 1) return;
        $('search_autocomplete').hide();
    },
    addResetButton: function(){
        this.searchField.insert({before:'<input class="reset-button" type="button" value="x"/>'});
        var obj = this;
        this.rb = $$('.reset-button').first();
        var offset = obj.searchField.offsetLeft + obj.searchField.offsetWidth - 15;
        if($$('div.xsearch-select').length > 0){
            offset-=15;
        }
        this.rb.style.left = offset + 'px'; 
        this.rb.observe('click',function(){
            obj.stopRequest();
            obj.isValid = false;
            obj.searchField.value = '';
            obj.onBlur();
            obj.resetButton();
        });
        this.resetButton();
    },
    resetButton: function(enable){
        if(enable && enable == true){
            this.rb.show();
        } else {
            this.rb.hide();
        }
    }
});