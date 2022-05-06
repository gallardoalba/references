<!DOCTYPE HTML>
<html>
<head>
<title>JabRef references</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript">
<!--
// QuickSearch script for JabRef HTML export 
// Version: 3.0
//
// Copyright (c) 2006-2011, Mark Schenk
//
// This software is distributed under a Creative Commons Attribution 3.0 License
// http://creativecommons.org/licenses/by/3.0/
//
// Features:
// - intuitive find-as-you-type searching
//    ~ case insensitive
//    ~ ignore diacritics (optional)
//
// - search with/without Regular Expressions
// - match BibTeX key
//

// Search settings
var searchAbstract = true;	// search in abstract
var searchReview = true;	// search in review

var noSquiggles = true; 	// ignore diacritics when searching
var searchRegExp = false; 	// enable RegExp searches


if (window.addEventListener) {
	window.addEventListener("load",initSearch,false); }
else if (window.attachEvent) {
	window.attachEvent("onload", initSearch); }

function initSearch() {
	// check for quick search table and searchfield
	if (!document.getElementById('qs_table')||!document.getElementById('quicksearch')) { return; }

	// load all the rows and sort into arrays
	loadTableData();
	
	//find the query field
	qsfield = document.getElementById('qs_field');

	// previous search term; used for speed optimisation
	prevSearch = '';

	//find statistics location
	stats = document.getElementById('stat');
	setStatistics(-1);
	
	// set up preferences
	initPreferences();

	// shows the searchfield
	document.getElementById('quicksearch').style.display = 'block';
	document.getElementById('qs_field').onkeyup = quickSearch;
}

function loadTableData() {
	// find table and appropriate rows
	searchTable = document.getElementById('qs_table');
	var allRows = searchTable.getElementsByTagName('tbody')[0].getElementsByTagName('tr');

	// split all rows into entryRows and infoRows (e.g. abstract, review, bibtex)
	entryRows = new Array(); infoRows = new Array(); absRows = new Array(); revRows = new Array();

	// get data from each row
	entryRowsData = new Array(); absRowsData = new Array(); revRowsData = new Array(); 
	
	BibTeXKeys = new Array();
	
	for (var i=0, k=0, j=0; i<allRows.length;i++) {
		if (allRows[i].className.match(/entry/)) {
			entryRows[j] = allRows[i];
			entryRowsData[j] = stripDiacritics(getTextContent(allRows[i]));
			allRows[i].id ? BibTeXKeys[j] = allRows[i].id : allRows[i].id = 'autokey_'+j;
			j ++;
		} else {
			infoRows[k++] = allRows[i];
			// check for abstract/review
			if (allRows[i].className.match(/abstract/)) {
				absRows.push(allRows[i]);
				absRowsData[j-1] = stripDiacritics(getTextContent(allRows[i]));
			} else if (allRows[i].className.match(/review/)) {
				revRows.push(allRows[i]);
				revRowsData[j-1] = stripDiacritics(getTextContent(allRows[i]));
			}
		}
	}
	//number of entries and rows
	numEntries = entryRows.length;
	numInfo = infoRows.length;
	numAbs = absRows.length;
	numRev = revRows.length;
}

function quickSearch(){
	
	tInput = qsfield;

	if (tInput.value.length == 0) {
		showAll();
		setStatistics(-1);
		qsfield.className = '';
		return;
	} else {
		t = stripDiacritics(tInput.value);

		if(!searchRegExp) { t = escapeRegExp(t); }
			
		// only search for valid RegExp
		try {
			textRegExp = new RegExp(t,"i");
			closeAllInfo();
			qsfield.className = '';
		}
			catch(err) {
			prevSearch = tInput.value;
			qsfield.className = 'invalidsearch';
			return;
		}
	}
	
	// count number of hits
	var hits = 0;

	// start looping through all entry rows
	for (var i = 0; cRow = entryRows[i]; i++){

		// only show search the cells if it isn't already hidden OR if the search term is getting shorter, then search all
		if(cRow.className.indexOf('noshow')==-1 || tInput.value.length <= prevSearch.length){
			var found = false; 

			if (entryRowsData[i].search(textRegExp) != -1 || BibTeXKeys[i].search(textRegExp) != -1){ 
				found = true;
			} else {
				if(searchAbstract && absRowsData[i]!=undefined) {
					if (absRowsData[i].search(textRegExp) != -1){ found=true; } 
				}
				if(searchReview && revRowsData[i]!=undefined) {
					if (revRowsData[i].search(textRegExp) != -1){ found=true; } 
				}
			}
			
			if (found){
				cRow.className = 'entry show';
				hits++;
			} else {
				cRow.className = 'entry noshow';
			}
		}
	}

	// update statistics
	setStatistics(hits)
	
	// set previous search value
	prevSearch = tInput.value;
}


// Strip Diacritics from text
// http://stackoverflow.com/questions/990904/javascript-remove-accents-in-strings

// String containing replacement characters for stripping accents 
var stripstring = 
    'AAAAAAACEEEEIIII'+
    'DNOOOOO.OUUUUY..'+
    'aaaaaaaceeeeiiii'+
    'dnooooo.ouuuuy.y'+
    'AaAaAaCcCcCcCcDd'+
    'DdEeEeEeEeEeGgGg'+
    'GgGgHhHhIiIiIiIi'+
    'IiIiJjKkkLlLlLlL'+
    'lJlNnNnNnnNnOoOo'+
    'OoOoRrRrRrSsSsSs'+
    'SsTtTtTtUuUuUuUu'+
    'UuUuWwYyYZzZzZz.';

function stripDiacritics(str){

    if(noSquiggles==false){
        return str;
    }

    var answer='';
    for(var i=0;i<str.length;i++){
        var ch=str[i];
        var chindex=ch.charCodeAt(0)-192;   // Index of character code in the strip string
        if(chindex>=0 && chindex<stripstring.length){
            // Character is within our table, so we can strip the accent...
            var outch=stripstring.charAt(chindex);
            // ...unless it was shown as a '.'
            if(outch!='.')ch=outch;
        }
        answer+=ch;
    }
    return answer;
}

// http://stackoverflow.com/questions/3446170/escape-string-for-use-in-javascript-regex
// NOTE: must escape every \ in the export code because of the JabRef Export...
function escapeRegExp(str) {
  return str.replace(/[-\[\]\/\{\}\(\)\*\+\?\.\\\^\$\|]/g, "\\$&");
}

function toggleInfo(articleid,info) {

	var entry = document.getElementById(articleid);
	var abs = document.getElementById('abs_'+articleid);
	var rev = document.getElementById('rev_'+articleid);
	var bib = document.getElementById('bib_'+articleid);
	
	if (abs && info == 'abstract') {
		abs.className.indexOf('noshow') == -1?abs.className = 'abstract noshow':abs.className = 'abstract show';
	} else if (rev && info == 'review') {
		rev.className.indexOf('noshow') == -1?rev.className = 'review noshow':rev.className = 'review show';
	} else if (bib && info == 'bibtex') {
		bib.className.indexOf('noshow') == -1?bib.className = 'bibtex noshow':bib.className = 'bibtex show';
	} else { 
		return;
	}

	// check if one or the other is available
	var revshow; var absshow; var bibshow;
	(abs && abs.className.indexOf('noshow') == -1)? absshow = true: absshow = false;
	(rev && rev.className.indexOf('noshow') == -1)? revshow = true: revshow = false;	
	(bib && bib.className.indexOf('noshow') == -1)? bibshow = true: bibshow = false;
	
	// highlight original entry
	if(entry) {
		if (revshow || absshow || bibshow) {
		entry.className = 'entry highlight show';
		} else {
		entry.className = 'entry show';
		}
	}
	
	// When there's a combination of abstract/review/bibtex showing, need to add class for correct styling
	if(absshow) {
		(revshow||bibshow)?abs.className = 'abstract nextshow':abs.className = 'abstract';
	} 
	if (revshow) {
		bibshow?rev.className = 'review nextshow': rev.className = 'review';
	}	
	
}

function setStatistics (hits) {
	if(hits < 0) { hits=numEntries; }
	if(stats) { stats.firstChild.data = hits + '/' + numEntries}
}

function getTextContent(node) {
	// Function written by Arve Bersvendsen
	// http://www.virtuelvis.com
	
	if (node.nodeType == 3) {
	return node.nodeValue;
	} // text node
	if (node.nodeType == 1 && node.className != "infolinks") { // element node
	var text = [];
	for (var chld = node.firstChild;chld;chld=chld.nextSibling) {
		text.push(getTextContent(chld));
	}
	return text.join("");
	} return ""; // some other node, won't contain text nodes.
}

function showAll(){
	closeAllInfo();
	for (var i = 0; i < numEntries; i++){ entryRows[i].className = 'entry show'; }
}

function closeAllInfo(){
	for (var i=0; i < numInfo; i++){
		if (infoRows[i].className.indexOf('noshow') ==-1) {
			infoRows[i].className = infoRows[i].className + ' noshow';
		}
	}
}

function clearQS() {
	qsfield.value = '';
	showAll();
}

function redoQS(){
	showAll();
	quickSearch(qsfield);
}

function updateSetting(obj){
	var option = obj.id;
	var checked = obj.value;

	switch(option)
	 {
	 case "opt_searchAbs":
	   searchAbstract=!searchAbstract;
	   redoQS();
	   break;
	 case "opt_searchRev":
	   searchReview=!searchReview;
	   redoQS();
	   break;
	 case "opt_useRegExp":
	   searchRegExp=!searchRegExp;
	   redoQS();
	   break;
	 case "opt_noAccents":
	   noSquiggles=!noSquiggles;
	   loadTableData();
	   redoQS();
	   break;
	 }
}

function initPreferences(){
	if(searchAbstract){document.getElementById("opt_searchAbs").checked = true;}
	if(searchReview){document.getElementById("opt_searchRev").checked = true;}
	if(noSquiggles){document.getElementById("opt_noAccents").checked = true;}
	if(searchRegExp){document.getElementById("opt_useRegExp").checked = true;}
	
	if(numAbs==0) {document.getElementById("opt_searchAbs").parentNode.style.display = 'none';}
	if(numRev==0) {document.getElementById("opt_searchRev").parentNode.style.display = 'none';}	
}

function toggleSettings(){
	var togglebutton = document.getElementById('showsettings');
	var settings = document.getElementById('settings');
	
	if(settings.className == "hidden"){
		settings.className = "show";
		togglebutton.innerText = "close settings";
		togglebutton.textContent = "close settings";
	}else{
		settings.className = "hidden";
		togglebutton.innerText = "settings...";		
		togglebutton.textContent = "settings...";
	}
}

-->
</script>
<style type="text/css">
body { background-color: white; font-family: Arial, sans-serif; font-size: 13px; line-height: 1.2; padding: 1em; color: #2E2E2E; width: 50em; margin: auto auto; }

form#quicksearch { width: auto; border-style: solid; border-color: gray; border-width: 1px 0px; padding: 0.7em 0.5em; display:none; position:relative; }
span#searchstat {padding-left: 1em;}

div#settings { margin-top:0.7em; /* border-bottom: 1px transparent solid; background-color: #efefef; border: 1px grey solid; */ }
div#settings ul {margin: 0; padding: 0; }
div#settings li {margin: 0; padding: 0 1em 0 0; display: inline; list-style: none; }
div#settings li + li { border-left: 2px #efefef solid; padding-left: 0.5em;}
div#settings input { margin-bottom: 0px;}

div#settings.hidden {display:none;}

#showsettings { border: 1px grey solid; padding: 0 0.5em; float:right; line-height: 1.6em; text-align: right; }
#showsettings:hover { cursor: pointer; }

.invalidsearch { background-color: red; }
input[type="button"] { background-color: #efefef; border: 1px #2E2E2E solid;}

table { border: 1px gray none; width: 100%; empty-cells: show; border-spacing: 0em 0.1em; margin: 1em 0em; }
th, td { border: none; padding: 0.5em; vertical-align: top; text-align: justify; }

td a { color: navy; text-decoration: none; }
td a:hover  { text-decoration: underline; }

tr.noshow { display: none;}
tr.highlight td { background-color: #EFEFEF; border-top: 2px #2E2E2E solid; font-weight: bold; }
tr.abstract td, tr.review td, tr.bibtex td { background-color: #EFEFEF; text-align: justify; border-bottom: 2px #2E2E2E solid; }
tr.nextshow td { border-bottom-style: none; }

tr.bibtex pre { width: 100%; overflow: auto; white-space: pre-wrap;}
p.infolinks { margin: 0.3em 0em 0em 0em; padding: 0px; }

@media print {
	p.infolinks, #qs_settings, #quicksearch, t.bibtex { display: none !important; }
	tr { page-break-inside: avoid; }
}
</style>
</head>
<body>

<form action="" id="quicksearch">
<input type="text" id="qs_field" autocomplete="off" placeholder="Type to search..." /> <input type="button" onclick="clearQS()" value="clear" />
<span id="searchstat">Matching entries: <span id="stat">0</span></span>
<div id="showsettings" onclick="toggleSettings()">settings...</div>
<div id="settings" class="hidden">
<ul>
<li><input type="checkbox" class="search_setting" id="opt_searchAbs" onchange="updateSetting(this)"><label for="opt_searchAbs"> include abstract</label></li>
<li><input type="checkbox" class="search_setting" id="opt_searchRev" onchange="updateSetting(this)"><label for="opt_searchRev"> include review</label></li>
<li><input type="checkbox" class="search_setting" id="opt_useRegExp" onchange="updateSetting(this)"><label for="opt_useRegExp"> use RegExp</label></li>
<li><input type="checkbox" class="search_setting" id="opt_noAccents" onchange="updateSetting(this)"><label for="opt_noAccents"> ignore accents</label></li>
</ul>
</div>
</form>
<table id="qs_table" border="1">
<tbody>
<tr id="Alsheikh2022" class="entry">
	<td>Alsheikh AJ, Wollenhaupt S, King EA, Reeb J, Ghosh S, Stolzenburg LR, Tamim S, Lazar J, Davis JW and Jacob HJ (2022), <i>"The landscape of GWAS validation; systematic review identifying 309 validated non-coding variants across 130 human diseases"</i>, BMC Medical Genomics 2022 15:1., apr, 2022.  Vol. 15(1), pp. 1-21. BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('Alsheikh2022','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Alsheikh2022','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/S12920-022-01216-W" target="_blank">DOI</a>] [<a href="https://link.springer.com/articles/10.1186/s12920-022-01216-w https://link.springer.com/article/10.1186/s12920-022-01216-w" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Alsheikh2022" class="abstract noshow">
	<td><b>Abstract</b>: The remarkable growth of genome-wide association studies (GWAS) has created a critical need to experimentally validate the disease-associated variants, 90% of which involve non-coding variants. To determine how the field is addressing this urgent need, we performed a comprehensive literature review identifying 36,676 articles. These were reduced to 1454 articles through a set of filters using natural language processing and ontology-based text-mining. This was followed by manual curation and cross-referencing against the GWAS catalog, yielding a final set of 286 articles. We identified 309 experimentally validated non-coding GWAS variants, regulating 252 genes across 130 human disease traits. These variants covered a variety of regulatory mechanisms. Interestingly, 70% (215/309) acted through cis-regulatory elements, with the remaining through promoters (22%, 70/309) or non-coding RNAs (8%, 24/309). Several validation approaches were utilized in these studies, including gene expression (n = 272), transcription factor binding (n = 175), reporter assays (n = 171), in vivo models (n = 104), genome editing (n = 96) and chromatin interaction (n = 33). This review of the literature is the first to systematically evaluate the status and the landscape of experimentation being used to validate non-coding GWAS-identified variants. Our results clearly underscore the multifaceted approach needed for experimental validation, have practical implications on variant prioritization and considerations of target gene nomination. While the field has a long way to go to validate the thousands of GWAS associations, we show that progress is being made and provide exemplars of validation studies covering a wide variety of mechanisms, target genes, and disease areas.</td>
</tr>
<tr id="bib_Alsheikh2022" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Alsheikh2022,
  author = {Alsheikh, Ammar J. and Wollenhaupt, Sabrina and King, Emily A. and Reeb, Jonas and Ghosh, Sujana and Stolzenburg, Lindsay R. and Tamim, Saleh and Lazar, Jozef and Davis, J. Wade and Jacob, Howard J.},
  title = {The landscape of GWAS validation; systematic review identifying 309 validated non-coding variants across 130 human diseases},
  journal = {BMC Medical Genomics 2022 15:1},
  publisher = {BioMed Central},
  year = {2022},
  volume = {15},
  number = {1},
  pages = {1--21},
  url = {https://link.springer.com/articles/10.1186/s12920-022-01216-w https://link.springer.com/article/10.1186/s12920-022-01216-w},
  doi = {10.1186/S12920-022-01216-W}
}
</pre></td>
</tr>
<tr id="Altshuler2008" class="entry">
	<td>Altshuler D, Daly MJ and Lander ES (2008), <i>"Genetic Mapping in Human Disease"</i>, Science (New York, N.Y.)., nov, 2008.  Vol. 322(5903), pp. 881. NIH Public Access.
	<p class="infolinks">[<a href="javascript:toggleInfo('Altshuler2008','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Altshuler2008','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1126/SCIENCE.1156409" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC2694957/ /pmc/articles/PMC2694957/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2694957/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Altshuler2008" class="abstract noshow">
	<td><b>Abstract</b>: Genetic mapping provides a powerful approach to identify genes and biological processes underlying any trait influenced by inheritance, including human diseases. We discuss the intellectual foundations of genetic mapping of Mendelian and complex traits in humans, examine lessons emerging from linkage analysis of Mendelian diseases and genome-wide association studies of common diseases, and discuss questions and challenges that lie ahead.</td>
</tr>
<tr id="bib_Altshuler2008" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Altshuler2008,
  author = {Altshuler, David and Daly, Mark J. and Lander, Eric S.},
  title = {Genetic Mapping in Human Disease},
  journal = {Science (New York, N.Y.)},
  publisher = {NIH Public Access},
  year = {2008},
  volume = {322},
  number = {5903},
  pages = {881},
  url = {/pmc/articles/PMC2694957/ /pmc/articles/PMC2694957/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2694957/},
  doi = {10.1126/SCIENCE.1156409}
}
</pre></td>
</tr>
<tr id="Antonarakis2006" class="entry">
	<td>Antonarakis SE and Beckmann JS (2006), <i>"Mendelian disorders deserve more attention"</i>, Nature Reviews Genetics.  Vol. 7(4), pp. 277-282.
	<p class="infolinks">[<a href="javascript:toggleInfo('Antonarakis2006','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Antonarakis2006','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1038/nrg1826" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Antonarakis2006" class="abstract noshow">
	<td><b>Abstract</b>: The study of inherited monogenic diseases has contributed greatly to our mechanistic understanding of pathogenic mutations and gene regulation, and to the development of effective diagnostic tools. But interest has gradually shifted away from monogenic diseases, which collectively affect only a small fraction of the world's population, towards multifactorial, common diseases. The quest for the genetic variability associated with common traits should not be done at the expense of Mendelian disorders, because the latter could still contribute greatly to understanding the aetiology of complex traits. textcopyright 2006 Nature Publishing Group.</td>
</tr>
<tr id="bib_Antonarakis2006" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Antonarakis2006,
  author = {Antonarakis, Stylianos E. and Beckmann, Jacques S.},
  title = {Mendelian disorders deserve more attention},
  journal = {Nature Reviews Genetics},
  year = {2006},
  volume = {7},
  number = {4},
  pages = {277--282},
  doi = {10.1038/nrg1826}
}
</pre></td>
</tr>
<tr id="Arabfard2019" class="entry">
	<td>Arabfard M, Ohadi M, Rezaei Tabar V, Delbari A and Kavousi K (2019), <i>"Genome-wide prediction and prioritization of human aging genes by data fusion: A machine learning approach"</i>, BMC Genomics., nov, 2019.  Vol. 20(1), pp. 1-13. BioMed Central Ltd..
	<p class="infolinks">[<a href="javascript:toggleInfo('Arabfard2019','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Arabfard2019','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/S12864-019-6140-0/TABLES/8" target="_blank">DOI</a>] [<a href="https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-019-6140-0" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Arabfard2019" class="abstract noshow">
	<td><b>Abstract</b>: Background: Machine learning can effectively nominate novel genes for various research purposes in the laboratory. On a genome-wide scale, we implemented multiple databases and algorithms to predict and prioritize the human aging genes (PPHAGE). Results: We fused data from 11 databases, and used Na&iuml;ve Bayes classifier and positive unlabeled learning (PUL) methods, NB, Spy, and Rocchio-SVM, to rank human genes in respect with their implication in aging. The PUL methods enabled us to identify a list of negative (non-aging) genes to use alongside the seed (known age-related) genes in the ranking process. Comparison of the PUL algorithms revealed that none of the methods for identifying a negative sample were advantageous over other methods, and their simultaneous use in a form of fusion was critical for obtaining optimal results (PPHAGE is publicly available at https://cbb.ut.ac.ir/pphage). Conclusion: We predict and prioritize over 3,000 candidate age-related genes in human, based on significant ranking scores. The identified candidate genes are associated with pathways, ontologies, and diseases that are linked to aging, such as cancer and diabetes. Our data offer a platform for future experimental research on the genetic and biological aspects of aging. Additionally, we demonstrate that fusion of PUL methods and data sources can be successfully used for aging and disease candidate gene prioritization.</td>
</tr>
<tr id="bib_Arabfard2019" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Arabfard2019,
  author = {Arabfard, Masoud and Ohadi, Mina and Rezaei Tabar, Vahid and Delbari, Ahmad and Kavousi, Kaveh},
  title = {Genome-wide prediction and prioritization of human aging genes by data fusion: A machine learning approach},
  journal = {BMC Genomics},
  publisher = {BioMed Central Ltd.},
  year = {2019},
  volume = {20},
  number = {1},
  pages = {1--13},
  url = {https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-019-6140-0},
  doi = {10.1186/S12864-019-6140-0/TABLES/8}
}
</pre></td>
</tr>
<tr id="Bertram2009" class="entry">
	<td>Bertram L and Tanzi RE (2009), <i>"Genome-wide association studies in Alzheimer's disease"</i>, Human Molecular Genetics., oct, 2009.  Vol. 18(R2), pp. R137. Oxford University Press.
	<p class="infolinks">[<a href="javascript:toggleInfo('Bertram2009','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Bertram2009','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1093/HMG/DDP406" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC2758713/ /pmc/articles/PMC2758713/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2758713/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Bertram2009" class="abstract noshow">
	<td><b>Abstract</b>: Genome-wide association studies (GWAS) have gained considerable momentum over the last couple of years for the identification of novel complex disease genes. In the field of Alzheimer's disease (AD), there are currently eight published and two provisionally reported GWAS, highlighting over two dozen novel potential susceptibility loci beyond the well-established APOE association. On the basis of the data available at the time of this writing, the most compelling novel GWAS signal has been observed in GAB2 (GRB2-associated binding protein 2), followed by less consistently replicated signals in galanin-like peptide (GALP), piggyBac transposable element derived 1 (PGBD1), tyrosine kinase, non-receptor 1 (TNK1). Furthermore, consistent replication has been recently announced for CLU (clusterin, also known as apolipoprotein J). Finally, there are at least three replicated loci in hitherto uncharacterized genomic intervals on chromosomes 14q32.13, 14q31.2 and 6q24.1 likely implicating the existence of novel AD genes in these regions. In this review, we will discuss the characteristics and potential relevance to pathogenesis of the outcomes of all currently available GWAS in AD. A particular emphasis will be laid on findings with independent data in favor of the original association. textcopyright The Author 2009. Published by Oxford University Press. All rights reserved.</td>
</tr>
<tr id="bib_Bertram2009" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Bertram2009,
  author = {Bertram, Lars and Tanzi, Rudolph E.},
  title = {Genome-wide association studies in Alzheimer's disease},
  journal = {Human Molecular Genetics},
  publisher = {Oxford University Press},
  year = {2009},
  volume = {18},
  number = {R2},
  pages = {R137},
  url = {/pmc/articles/PMC2758713/ /pmc/articles/PMC2758713/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2758713/},
  doi = {10.1093/HMG/DDP406}
}
</pre></td>
</tr>
<tr id="Broeckx2017" class="entry">
	<td>Broeckx BJ, Derrien T, Mottier S, Wucher V, Cadieu E, H&eacute;dan B, Le B&eacute;guec C, Botherel N, Lindblad-Toh K, Saunders JH, Deforce D, Andr&eacute; C, Peelman L and Hitte C (2017), <i>"An exome sequencing based approach for genome-wide association studies in the dog"</i>, Scientific Reports 2017 7:1., nov, 2017.  Vol. 7(1), pp. 1-11. Nature Publishing Group.
	<p class="infolinks">[<a href="javascript:toggleInfo('Broeckx2017','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Broeckx2017','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1038/s41598-017-15947-9" target="_blank">DOI</a>] [<a href="https://www.nature.com/articles/s41598-017-15947-9" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Broeckx2017" class="abstract noshow">
	<td><b>Abstract</b>: Genome-wide association studies (GWAS) are widely used to identify loci associated with phenotypic traits in the domestic dog that has emerged as a model for Mendelian and complex traits. However, a disadvantage of GWAS is that it always requires subsequent fine-mapping or sequencing to pinpoint causal mutations. Here, we performed whole exome sequencing (WES) and canine high-density (cHD) SNP genotyping of 28 dogs from 3 breeds to compare the SNP and linkage disequilibrium characteristics together with the power and mapping precision of exome-guided GWAS (EG-GWAS) versus cHD-based GWAS. Using simulated phenotypes, we showed that EG-GWAS has a higher power than cHD to detect associations within target regions and less power outside target regions, with power being influenced further by sample size and SNP density. We analyzed two real phenotypes (hair length and furnishing), that are fixed in certain breeds to characterize mapping precision of the known causal mutations. EG-GWAS identified the associated exonic and 3′UTR variants within the FGF5 and RSPO2 genes, respectively, with only a few samples per breed. In conclusion, we demonstrated that EG-GWAS can identify loci associated with Mendelian phenotypes both within and across breeds.</td>
</tr>
<tr id="bib_Broeckx2017" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Broeckx2017,
  author = {Broeckx, Bart J.G. and Derrien, Thomas and Mottier, St&eacute;phanie and Wucher, Valentin and Cadieu, Edouard and H&eacute;dan, Beno&icirc;t and Le B&eacute;guec, C&eacute;line and Botherel, Nadine and Lindblad-Toh, Kerstin and Saunders, Jimmy H. and Deforce, Dieter and Andr&eacute;, Catherine and Peelman, Luc and Hitte, Christophe},
  title = {An exome sequencing based approach for genome-wide association studies in the dog},
  journal = {Scientific Reports 2017 7:1},
  publisher = {Nature Publishing Group},
  year = {2017},
  volume = {7},
  number = {1},
  pages = {1--11},
  url = {https://www.nature.com/articles/s41598-017-15947-9},
  doi = {10.1038/s41598-017-15947-9}
}
</pre></td>
</tr>
<tr id="Broekema2020" class="entry">
	<td>Broekema RV, Bakker OB and Jonkers IH (2020), <i>"A practical view of fine-mapping and gene prioritization in the post-genome-wide association era"</i>, Open Biology.  Vol. 10(1)
	<p class="infolinks">[<a href="javascript:toggleInfo('Broekema2020','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Broekema2020','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1098/rsob.190221" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Broekema2020" class="abstract noshow">
	<td><b>Abstract</b>: Over the past 15 years, genome-wide association studies (GWASs) have enabled the systematic identification of genetic loci associated with traits and diseases. However, due to resolution issues and methodological limitations, the true causal variants and genes associated with traits remain difficult to identify. In this post-GWAS era, many biological and computational fine-mapping approaches now aim to solve these issues. Here, we review fine-mapping and gene prioritization approaches that, when combined, will improve the understanding of the underlying mechanisms of complex traits and diseases. Fine-mapping of genetic variants has become increasingly sophisticated: initially, variants were simply overlapped with functional elements, but now the impact of variants on regulatory activity and direct variant-gene 3D interactions can be identified. Moreover, gene manipulation by CRISPR/Cas9, the identification of expression quantitative trait loci and the use of co-expression networks have all increased our understanding of the genes and pathways affected by GWAS loci. However, despite this progress, limitations including the lack of cell-type- and disease-specific data and the ever-increasing complexity of polygenic models of traits pose serious challenges. Indeed, the combination of fine-mapping and gene prioritization by statistical, functional and population-based strategies will be necessary to truly understand how GWAS loci contribute to complex traits and diseases.</td>
</tr>
<tr id="bib_Broekema2020" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Broekema2020,
  author = {Broekema, R. V. and Bakker, O. B. and Jonkers, I. H.},
  title = {A practical view of fine-mapping and gene prioritization in the post-genome-wide association era},
  journal = {Open Biology},
  year = {2020},
  volume = {10},
  number = {1},
  doi = {10.1098/rsob.190221}
}
</pre></td>
</tr>
<tr id="Bromberg2013" class="entry">
	<td>Bromberg Y (2013), <i>"Chapter 15: Disease Gene Prioritization"</i>, PLoS Computational Biology., apr, 2013.  Vol. 9(4) Public Library of Science.
	<p class="infolinks">[<a href="javascript:toggleInfo('Bromberg2013','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Bromberg2013','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1371/JOURNAL.PCBI.1002902" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC3635969/ /pmc/articles/PMC3635969/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3635969/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Bromberg2013" class="abstract noshow">
	<td><b>Abstract</b>: Disease-causing aberrations in the normal function of a gene define that gene as a disease gene. Proving a causal link between a gene and a disease experimentally is expensive and time-consuming. Comprehensive prioritization of candidate genes prior to experimental testing drastically reduces the associated costs. Computational gene prioritization is based on various pieces of correlative evidence that associate each gene with the given disease and suggest possible causal links. A fair amount of this evidence comes from high-throughput experimentation. Thus, well-developed methods are necessary to reliably deal with the quantity of information at hand. Existing gene prioritization techniques already significantly improve the outcomes of targeted experimental studies. Faster and more reliable techniques that account for novel data types are necessary for the development of new diagnostics, treatments, and cure for many diseases. textcopyright 2013 Yana Bromberg.</td>
</tr>
<tr id="bib_Bromberg2013" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Bromberg2013,
  author = {Bromberg, Yana},
  title = {Chapter 15: Disease Gene Prioritization},
  journal = {PLoS Computational Biology},
  publisher = {Public Library of Science},
  year = {2013},
  volume = {9},
  number = {4},
  url = {/pmc/articles/PMC3635969/ /pmc/articles/PMC3635969/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3635969/},
  doi = {10.1371/JOURNAL.PCBI.1002902}
}
</pre></td>
</tr>
<tr id="Brueggeman2020" class="entry">
	<td>Brueggeman L, Koomar T and Michaelson JJ (2020), <i>"Forecasting risk gene discovery in autism with machine learning and genome-scale data"</i>, Scientific Reports 2020 10:1., mar, 2020.  Vol. 10(1), pp. 1-11. Nature Publishing Group.
	<p class="infolinks">[<a href="javascript:toggleInfo('Brueggeman2020','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Brueggeman2020','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1038/s41598-020-61288-5" target="_blank">DOI</a>] [<a href="https://www.nature.com/articles/s41598-020-61288-5" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Brueggeman2020" class="abstract noshow">
	<td><b>Abstract</b>: Genetics has been one of the most powerful windows into the biology of autism spectrum disorder (ASD). It is estimated that a thousand or more genes may confer risk for ASD when functionally perturbed, however, only around 100 genes currently have sufficient evidence to be considered true “autism risk genes”. Massive genetic studies are currently underway producing data to implicate additional genes. This approach — although necessary — is costly and slow-moving, making identification of putative ASD risk genes with existing data vital. Here, we approach autism risk gene discovery as a machine learning problem, rather than a genetic association problem, by using genome-scale data as predictors to identify new genes with similar properties to established autism risk genes. This ensemble method, forecASD, integrates brain gene expression, heterogeneous network data, and previous gene-level predictors of autism association into an ensemble classifier that yields a single score indexing evidence of each gene's involvement in the etiology of autism. We demonstrate that forecASD has substantially better performance than previous predictors of autism association in three independent trio-based sequencing studies. Studying forecASD prioritized genes, we show that forecASD is a robust indicator of a gene's involvement in ASD etiology, with diverse applications to gene discovery, differential expression analysis, eQTL prioritization, and pathway enrichment analysis.</td>
</tr>
<tr id="bib_Brueggeman2020" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Brueggeman2020,
  author = {Brueggeman, Leo and Koomar, Tanner and Michaelson, Jacob J.},
  title = {Forecasting risk gene discovery in autism with machine learning and genome-scale data},
  journal = {Scientific Reports 2020 10:1},
  publisher = {Nature Publishing Group},
  year = {2020},
  volume = {10},
  number = {1},
  pages = {1--11},
  url = {https://www.nature.com/articles/s41598-020-61288-5},
  doi = {10.1038/s41598-020-61288-5}
}
</pre></td>
</tr>
<tr id="Deo2014" class="entry">
	<td>Deo RC, Musso G, Tasan M, Tang P, Poon A, Yuan C, Felix JF, Vasan RS, Beroukhim R, De Marco T, Kwok PY, MacRae CA and Roth FP (2014), <i>"Prioritizing causal disease genes using unbiased genomic features"</i>, Genome biology., dec, 2014.  Vol. 15(12), pp. 534. BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('Deo2014','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Deo2014','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/S13059-014-0534-8/FIGURES/5" target="_blank">DOI</a>] [<a href="https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0534-8" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Deo2014" class="abstract noshow">
	<td><b>Abstract</b>: BACKGROUND: Cardiovascular disease (CVD) is the leading cause of death in the developed world. Human genetic studies, including genome-wide sequencing and SNP-array approaches, promise to reveal disease genes and mechanisms representing new therapeutic targets. In practice, however, identification of the actual genes contributing to disease pathogenesis has lagged behind identification of associated loci, thus limiting the clinical benefits.<br>RESULTS: To aid in localizing causal genes, we develop a machine learning approach, Objective Prioritization for Enhanced Novelty (OPEN), which quantitatively prioritizes gene-disease associations based on a diverse group of genomic features. This approach uses only unbiased predictive features and thus is not hampered by a preference towards previously well-characterized genes. We demonstrate success in identifying genetic determinants for CVD-related traits, including cholesterol levels, blood pressure, and conduction system and cardiomyopathy phenotypes. Using OPEN, we prioritize genes, including FLNC, for association with increased left ventricular diameter, which is a defining feature of a prevalent cardiovascular disorder, dilated cardiomyopathy or DCM. Using a zebrafish model, we experimentally validate FLNC and identify a novel FLNC splice-site mutation in a patient with severe DCM.<br>CONCLUSION: Our approach stands to assist interpretation of large-scale genetic studies without compromising their fundamentally unbiased nature.</td>
</tr>
<tr id="bib_Deo2014" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Deo2014,
  author = {Deo, Rahul C. and Musso, Gabriel and Tasan, Murat and Tang, Paul and Poon, Annie and Yuan, Christiana and Felix, Janine F. and Vasan, Ramachandran S. and Beroukhim, Rameen and De Marco, Teresa and Kwok, Pui Yan and MacRae, Calum A. and Roth, Frederick P.},
  title = {Prioritizing causal disease genes using unbiased genomic features},
  journal = {Genome biology},
  publisher = {BioMed Central},
  year = {2014},
  volume = {15},
  number = {12},
  pages = {534},
  url = {https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0534-8},
  doi = {10.1186/S13059-014-0534-8/FIGURES/5}
}
</pre></td>
</tr>
<tr id="Ewis2005" class="entry">
	<td>Ewis AA, Zhelev Z, Bakalova R, Fukuoka S, Shinohara Y, Ishikawa M and Baba Y (2005), <i>"A history of microarrays in biomedicine"</i>, Expert Review of Molecular Diagnostics.  Vol. 5(3), pp. 315-328.
	<p class="infolinks">[<a href="javascript:toggleInfo('Ewis2005','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Ewis2005','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1586/14737159.5.3.315" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Ewis2005" class="abstract noshow">
	<td><b>Abstract</b>: The fundamental strategy of the current postgenomic era or the era of functional genomics is to expand the scale of biologic research from studying single genes or proteins to studying all genes or proteins simultaneously using a systematic approach. As recently developed methods for obtaining genome-wide mRNA expression data, oligonucleotide and DNA microarrays are particularly powerful in the context of knowing the entire genome sequence and can provide a global view of changes in gene expression patterns in response to physiologic alterations or manipulation of transcriptional regulators. In biomedical research, such an approach will ultimately determine biologic behavior of both normal and diseased tissues, which may provide insights into disease mechanisms and identify novel markers and candidates for diagnostic, prognostic and therapeutic intervention. However, microarray technology is still in a continuous state of evolution and development, and it may take time to implement microarrays as a routine medical device. Many limitations exist and many challenges remain to be achieved to help inclusion of microarrays in clinical medicine. In this review, a brief history of microarrays in biomedical research is provided, including experimental overview, limitations, challenges and future developments. textcopyright 2005 Future Drugs Ltd.</td>
</tr>
<tr id="bib_Ewis2005" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Ewis2005,
  author = {Ewis, Ashraf A. and Zhelev, Zhivko and Bakalova, Rumiana and Fukuoka, Satoshi and Shinohara, Yasuo and Ishikawa, Mitsuru and Baba, Yoshinobu},
  title = {A history of microarrays in biomedicine},
  journal = {Expert Review of Molecular Diagnostics},
  year = {2005},
  volume = {5},
  number = {3},
  pages = {315--328},
  doi = {10.1586/14737159.5.3.315}
}
</pre></td>
</tr>
<tr id="Frayling2014" class="entry">
	<td>Frayling TM (2014), <i>"Genome-wide association studies: the good, the bad and the ugly"</i>, Clinical Medicine., aug, 2014.  Vol. 14(4), pp. 428. Royal College of Physicians.
	<p class="infolinks">[<a href="javascript:toggleInfo('Frayling2014','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Frayling2014','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.7861/CLINMEDICINE.14-4-428" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC4952840/ https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4952840/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Frayling2014" class="abstract noshow">
	<td><b>Abstract</b>: Before 2007 the number of common genetic variants reproducibly associated with common diseases and traits was fewer than 20. There are now many hundreds of variants reliably associated with all types of diseases and traits, from male pattern baldness to height to common disease predisposition, including metabolic disease, autoimmune disease and germline predisposition to cancer. Despite this success at identifying variants, the GWAS findings are not generally clinically useful to individual patients. Instead they represent a first step towards improved understanding of disease aetiology. textcopyright Royal College of Physicians 2014. All rights reserved.</td>
</tr>
<tr id="bib_Frayling2014" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Frayling2014,
  author = {Frayling, T. M.},
  title = {Genome-wide association studies: the good, the bad and the ugly},
  journal = {Clinical Medicine},
  publisher = {Royal College of Physicians},
  year = {2014},
  volume = {14},
  number = {4},
  pages = {428},
  url = {/pmc/articles/PMC4952840/ https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4952840/},
  doi = {10.7861/CLINMEDICINE.14-4-428}
}
</pre></td>
</tr>
<tr id="Gaare2020" class="entry">
	<td>Gaare JJ, Nido G, D&ouml;lle C, Sztromwasser P, Alves G, Tysnes OB, Haugarvoll K and Tzoulis C (2020), <i>"Meta-analysis of whole-exome sequencing data from two independent cohorts finds no evidence for rare variant enrichment in Parkinson disease associated loci"</i>, PLOS ONE., oct, 2020.  Vol. 15(10), pp. e0239824. Public Library of Science.
	<p class="infolinks">[<a href="javascript:toggleInfo('Gaare2020','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Gaare2020','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1371/JOURNAL.PONE.0239824" target="_blank">DOI</a>] [<a href="https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0239824" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Gaare2020" class="abstract noshow">
	<td><b>Abstract</b>: Parkinson disease (PD) is a complex neurodegenerative disorder influenced by both environmental and genetic factors. While genome wide association studies have identified several susceptibility loci, many causal variants and genes underlying these associations remain undetermined. Identifying these is essential in order to gain mechanistic insight and identify biological pathways that may be targeted therapeutically. We hypothesized that gene-based enrichment of rare mutations is likely to be found within susceptibility loci for PD and may help identify causal genes. Whole-exome sequencing data from two independent cohorts were analyzed in tandem and by meta-analysis and a third cohort genotyped using the NeuroX-array was used for replication analysis. We employed collapsing methods (burden and the sequence kernel association test) to detect gene-based enrichment of rare, protein-altering variation within established PD susceptibility loci. Our analyses showed trends for three genes (GALC, PARP9 and SEC23IP), but none of these survived multiple testing correction. Our findings provide no evidence of rare mutation enrichment in genes within PD-associated loci, in our datasets. While not excluding that rare mutations in these genes may influence the risk of idiopathic PD, our results suggest that, if such effects exist, much larger sequencing datasets will be required for their detection.</td>
</tr>
<tr id="bib_Gaare2020" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Gaare2020,
  author = {Gaare, Johannes Jernqvist and Nido, Gonzalo and D&ouml;lle, Christian and Sztromwasser, Pawe&lstrok; and Alves, Guido and Tysnes, Ole Bj&oslash;rn and Haugarvoll, Kristoffer and Tzoulis, Charalampos},
  title = {Meta-analysis of whole-exome sequencing data from two independent cohorts finds no evidence for rare variant enrichment in Parkinson disease associated loci},
  journal = {PLOS ONE},
  publisher = {Public Library of Science},
  year = {2020},
  volume = {15},
  number = {10},
  pages = {e0239824},
  url = {https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0239824},
  doi = {10.1371/JOURNAL.PONE.0239824}
}
</pre></td>
</tr>
<tr id="Gillis2012" class="entry">
	<td>Gillis J and Pavlidis P (2012), <i>"“Guilt by Association” Is the Exception Rather Than the Rule in Gene Networks"</i>, PLOS Computational Biology., mar, 2012.  Vol. 8(3), pp. e1002444. Public Library of Science.
	<p class="infolinks">[<a href="javascript:toggleInfo('Gillis2012','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Gillis2012','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1371/JOURNAL.PCBI.1002444" target="_blank">DOI</a>] [<a href="https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002444" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Gillis2012" class="abstract noshow">
	<td><b>Abstract</b>: Gene networks are commonly interpreted as encoding functional information in their connections. An extensively validated principle called guilt by association states that genes which are associated or interacting are more likely to share function. Guilt by association provides the central top-down principle for analyzing gene networks in functional terms or assessing their quality in encoding functional information. In this work, we show that functional information within gene networks is typically concentrated in only a very few interactions whose properties cannot be reliably related to the rest of the network. In effect, the apparent encoding of function within networks has been largely driven by outliers whose behaviour cannot even be generalized to individual genes, let alone to the network at large. While experimentalist-driven analysis of interactions may use prior expert knowledge to focus on the small fraction of critically important data, large-scale computational analyses have typically assumed that high-performance cross-validation in a network is due to a generalizable encoding of function. Because we find that gene function is not systemically encoded in networks, but dependent on specific and critical interactions, we conclude it is necessary to focus on the details of how networks encode function and what information computational analyses use to extract functional meaning. We explore a number of consequences of this and find that network structure itself provides clues as to which connections are critical and that systemic properties, such as scale-free-like behaviour, do not map onto the functional connectivity within networks.</td>
</tr>
<tr id="bib_Gillis2012" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Gillis2012,
  author = {Gillis, Jesse and Pavlidis, Paul},
  title = {“Guilt by Association” Is the Exception Rather Than the Rule in Gene Networks},
  journal = {PLOS Computational Biology},
  publisher = {Public Library of Science},
  year = {2012},
  volume = {8},
  number = {3},
  pages = {e1002444},
  url = {https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002444},
  doi = {10.1371/JOURNAL.PCBI.1002444}
}
</pre></td>
</tr>
<tr id="Glaab2012" class="entry">
	<td>Glaab E, Bacardit J, Garibaldi JM and Krasnogor N (2012), <i>"Using Rule-Based Machine Learning for Candidate Disease Gene Prioritization and Sample Classification of Cancer Gene Expression Data"</i>, PLOS ONE., jul, 2012.  Vol. 7(7), pp. e39932. Public Library of Science.
	<p class="infolinks">[<a href="javascript:toggleInfo('Glaab2012','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Glaab2012','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1371/JOURNAL.PONE.0039932" target="_blank">DOI</a>] [<a href="https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0039932" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Glaab2012" class="abstract noshow">
	<td><b>Abstract</b>: Microarray data analysis has been shown to provide an effective tool for studying cancer and genetic diseases. Although classical machine learning techniques have successfully been applied to find informative genes and to predict class labels for new samples, common restrictions of microarray analysis such as small sample sizes, a large attribute space and high noise levels still limit its scientific and clinical applications. Increasing the interpretability of prediction models while retaining a high accuracy would help to exploit the information content in microarray data more effectively. For this purpose, we evaluate our rule-based evolutionary machine learning systems, BioHEL and GAssist, on three public microarray cancer datasets, obtaining simple rule-based models for sample classification. A comparison with other benchmark microarray sample classifiers based on three diverse feature selection algorithms suggests that these evolutionary learning techniques can compete with state-of-the-art methods like support vector machines. The obtained models reach accuracies above 90% in two-level external cross-validation, with the added value of facilitating interpretation by using only combinations of simple if-then-else rules. As a further benefit, a literature mining analysis reveals that prioritizations of informative genes extracted from BioHEL's classification rule sets can outperform gene rankings obtained from a conventional ensemble feature selection in terms of the pointwise mutual information between relevant disease terms and the standardized names of top-ranked genes.</td>
</tr>
<tr id="bib_Glaab2012" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Glaab2012,
  author = {Glaab, Enrico and Bacardit, Jaume and Garibaldi, Jonathan M. and Krasnogor, Natalio},
  title = {Using Rule-Based Machine Learning for Candidate Disease Gene Prioritization and Sample Classification of Cancer Gene Expression Data},
  journal = {PLOS ONE},
  publisher = {Public Library of Science},
  year = {2012},
  volume = {7},
  number = {7},
  pages = {e39932},
  url = {https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0039932},
  doi = {10.1371/JOURNAL.PONE.0039932}
}
</pre></td>
</tr>
<tr id="Gupta2021" class="entry">
	<td>Gupta C, Ramegowda V, Basu S and Pereira A (2021), <i>"Using Network-Based Machine Learning to Predict Transcription Factors Involved in Drought Resistance"</i>, Frontiers in Genetics., jun, 2021.  Vol. 12 Frontiers Media S.A..
	<p class="infolinks">[<a href="javascript:toggleInfo('Gupta2021','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Gupta2021','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.3389/FGENE.2021.652189/FULL" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC8264776/ /pmc/articles/PMC8264776/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8264776/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Gupta2021" class="abstract noshow">
	<td><b>Abstract</b>: Gene regulatory networks underpin stress response pathways in plants. However, parsing these networks to prioritize key genes underlying a particular trait is challenging. Here, we have built the Gene Regulation and Association Network (GRAiN) of rice (Oryza sativa). GRAiN is an interactive query-based web-platform that allows users to study functional relationships between transcription factors (TFs) and genetic modules underlying abiotic-stress responses. We built GRAiN by applying a combination of different network inference algorithms to publicly available gene expression data. We propose a supervised machine learning framework that complements GRAiN in prioritizing genes that regulate stress signal transduction and modulate gene expression under drought conditions. Our framework converts intricate network connectivity patterns of 2160 TFs into a single drought score. We observed that TFs with the highest drought scores define the functional, structural, and evolutionary characteristics of drought resistance in rice. Our approach accurately predicted the function of OsbHLH148 TF, which we validated using in vitro protein-DNA binding assays and mRNA sequencing loss-of-function mutants grown under control and drought stress conditions. Our network and the complementary machine learning strategy lends itself to predicting key regulatory genes underlying other agricultural traits and will assist in the genetic engineering of desirable rice varieties.</td>
</tr>
<tr id="bib_Gupta2021" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Gupta2021,
  author = {Gupta, Chirag and Ramegowda, Venkategowda and Basu, Supratim and Pereira, Andy},
  title = {Using Network-Based Machine Learning to Predict Transcription Factors Involved in Drought Resistance},
  journal = {Frontiers in Genetics},
  publisher = {Frontiers Media S.A.},
  year = {2021},
  volume = {12},
  url = {/pmc/articles/PMC8264776/ /pmc/articles/PMC8264776/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8264776/},
  doi = {10.3389/FGENE.2021.652189/FULL}
}
</pre></td>
</tr>
<tr id="Haldane1937" class="entry">
	<td>Haldane B (1937), <i>"The linkage between the genes for colour-blindness and haemophilia in man"</i>, Proceedings of the Royal Society of London. Series B - Biological Sciences., jul, 1937.  Vol. 123(831), pp. 119-150. The Royal Society.
	<p class="infolinks">[<a href="javascript:toggleInfo('Haldane1937','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Haldane1937','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1098/RSPB.1937.0046" target="_blank">DOI</a>] [<a href="https://royalsocietypublishing.org/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Haldane1937" class="abstract noshow">
	<td><b>Abstract</b>: It is well established that colour-blindness and haemophilia are due to sex-linked genes. These genes appear to manifest themselves in all males who carry them. In women the gene for haemophilia is probably always recessive, the cases of alleged haemophilia in heterozygous women being very doubtful. On the other hand, colour-blind women whose putative fathers are not colour-blind occur too frequently to be explained by illegitimacy (Bell 1926). So colour-blindness is probably not always recessive. On the other hand, women homozygous for the gene appear always to be colour-blind. No cases of incomplete recessiveness occur in the new pedigrees here presented. It will be assumed th at a woman who is not colour-blind is not homozygous for the gene for colour-blindness. There are two distinct forms of colour-blindness, namely protanopia (“red-blindness”) and deuteranopia (“green-blindness”). According to Waaler (1927) the genes determining them form a series of five allelomorphs with the normal gene, and those for protanomalia and deuteranomalia. Haldane (1935) suggested that there are at least two different allelomorphic genes for haemophilia.</td>
</tr>
<tr id="bib_Haldane1937" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Haldane1937,
  author = {Haldane, Bell},
  title = {The linkage between the genes for colour-blindness and haemophilia in man},
  journal = {Proceedings of the Royal Society of London. Series B - Biological Sciences},
  publisher = {The Royal Society},
  year = {1937},
  volume = {123},
  number = {831},
  pages = {119--150},
  url = {https://royalsocietypublishing.org/},
  doi = {10.1098/RSPB.1937.0046}
}
</pre></td>
</tr>
<tr id="Han2019" class="entry">
	<td>Han Y, Yang J, Qian X, Cheng WC, Liu SH, Hua X, Zhou L, Yang Y, Wu Q, Liu P and Lu Y (2019), <i>"DriverML: a machine learning algorithm for identifying driver genes in cancer sequencing studies"</i>, Nucleic Acids Research., may, 2019.  Vol. 47(8), pp. e45-e45. Oxford Academic.
	<p class="infolinks">[<a href="javascript:toggleInfo('Han2019','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Han2019','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1093/NAR/GKZ096" target="_blank">DOI</a>] [<a href="https://academic.oup.com/nar/article/47/8/e45/5324448" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Han2019" class="abstract noshow">
	<td><b>Abstract</b>: Although rapid progress has been made in computational approaches for prioritizing cancer driver genes, research is far from achieving the ultimate goal of discovering a complete catalog of genes truly associated with cancer. Driver gene lists predicted from these computational tools lack consistency and are prone to false positives. Here, we developed an approach (DriverML) integrating Rao's score test and supervised machine learning to identify cancer driver genes. The weight parameters in the score statistics quantified the functional impacts of mutations on the protein. To obtain optimized weight parameters, the score statistics of prior driver genes were maximized on pan-cancer training data. We conducted rigorous and unbiased benchmark analysis and comparisons of DriverML with 20 other existing tools in 31 independent datasets from The Cancer Genome Atlas (TCGA). Our comprehensive evaluations demonstrated that DriverML was robust and powerful among various datasets and outperformed the other tools with a better balance of precision and sensitivity. In vitro cell-based assays further proved the validity of the DriverML prediction of novel driver genes. In summary, DriverML uses an innovative, machine learning-based approach to prioritize cancer driver genes and provides dramatic improvements over currently existing methods. Its source code is available at https://github.com/HelloYiHan/DriverML.</td>
</tr>
<tr id="bib_Han2019" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Han2019,
  author = {Han, Yi and Yang, Juze and Qian, Xinyi and Cheng, Wei Chung and Liu, Shu Hsuan and Hua, Xing and Zhou, Liyuan and Yang, Yaning and Wu, Qingbiao and Liu, Pengyuan and Lu, Yan},
  title = {DriverML: a machine learning algorithm for identifying driver genes in cancer sequencing studies},
  journal = {Nucleic Acids Research},
  publisher = {Oxford Academic},
  year = {2019},
  volume = {47},
  number = {8},
  pages = {e45--e45},
  url = {https://academic.oup.com/nar/article/47/8/e45/5324448},
  doi = {10.1093/NAR/GKZ096}
}
</pre></td>
</tr>
<tr id="He2011" class="entry">
	<td>He C, Weeks DE, Buyske S, Abecasis GR, Stewart WC and Matise TC (2011), <i>"Enhanced genetic maps from family-based disease studies: Population-specific comparisons"</i>, BMC Medical Genetics., jan, 2011.  Vol. 12(1), pp. 1-14. BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('He2011','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('He2011','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/1471-2350-12-15/FIGURES/3" target="_blank">DOI</a>] [<a href="https://bmcmedgenet.biomedcentral.com/articles/10.1186/1471-2350-12-15" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_He2011" class="abstract noshow">
	<td><b>Abstract</b>: Background: Accurate genetic maps are required for successful and efficient linkage mapping of disease genes. However, most available genome-wide genetic maps were built using only small collections of pedigrees, and therefore have large sampling errors. A large set of genetic studies genotyped by the NHLBI Mammalian Genotyping Service (MGS) provide appropriate data for generating more accurate maps.Results: We collected a large sample of uncleaned genotype data for 461 markers generated by the MGS using the Weber screening sets 9 and 10. This collection includes genotypes for over 4,400 pedigrees containing over 17,000 genotyped individuals from different populations. We identified and cleaned numerous relationship and genotyping errors, as well as verified the marker orders. We used this dataset to test for population-specific genetic maps, and to re-estimate the genetic map distances with greater precision; standard errors for all intervals are provided. The map-interval sizes from the European (or European descent), Chinese, and Hispanic samples are in quite good agreement with each other. We found one map interval on chromosome 8p with a statistically significant size difference between the European and Chinese samples, and several map intervals with significant size differences between the African American and Chinese samples. When comparing Palauan with European samples, a statistically significant difference was detected at the telomeric region of chromosome 11p. Several significant differences were also identified between populations in chromosomal and genome lengths.Conclusions: Our new population-specific screening set maps can be used to improve the accuracy of disease-mapping studies. As a result of the large sample size, the average length of the 95% confidence interval (CI) for a 10 cM map interval is only 2.4 cM, which is considerably smaller than on previously published maps. textcopyright 2011 He et al; licensee BioMed Central Ltd.</td>
</tr>
<tr id="bib_He2011" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{He2011,
  author = {He, Chunsheng and Weeks, Daniel E. and Buyske, Steven and Abecasis, Goncalo R. and Stewart, William C. and Matise, Tara C.},
  title = {Enhanced genetic maps from family-based disease studies: Population-specific comparisons},
  journal = {BMC Medical Genetics},
  publisher = {BioMed Central},
  year = {2011},
  volume = {12},
  number = {1},
  pages = {1--14},
  url = {https://bmcmedgenet.biomedcentral.com/articles/10.1186/1471-2350-12-15},
  doi = {10.1186/1471-2350-12-15/FIGURES/3}
}
</pre></td>
</tr>
<tr id="Heather2016" class="entry">
	<td>Heather JM and Chain B (2016), <i>"The sequence of sequencers: The history of sequencing DNA"</i>, Genomics., jan, 2016.  Vol. 107(1), pp. 1. Elsevier.
	<p class="infolinks">[<a href="javascript:toggleInfo('Heather2016','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Heather2016','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1016/J.YGENO.2015.11.003" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC4727787/ /pmc/articles/PMC4727787/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4727787/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Heather2016" class="abstract noshow">
	<td><b>Abstract</b>: Determining the order of nucleic acid residues in biological samples is an integral component of a wide variety of research applications. Over the last fifty years large numbers of researchers have applied themselves to the production of techniques and technologies to facilitate this feat, sequencing DNA and RNA molecules. This time-scale has witnessed tremendous changes, moving from sequencing short oligonucleotides to millions of bases, from struggling towards the deduction of the coding sequence of a single gene to rapid and widely available whole genome sequencing. This article traverses those years, iterating through the different generations of sequencing technology, highlighting some of the key discoveries, researchers, and sequences along the way.</td>
</tr>
<tr id="bib_Heather2016" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Heather2016,
  author = {Heather, James M. and Chain, Benjamin},
  title = {The sequence of sequencers: The history of sequencing DNA},
  journal = {Genomics},
  publisher = {Elsevier},
  year = {2016},
  volume = {107},
  number = {1},
  pages = {1},
  url = {/pmc/articles/PMC4727787/ /pmc/articles/PMC4727787/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4727787/},
  doi = {10.1016/J.YGENO.2015.11.003}
}
</pre></td>
</tr>
<tr id="Heller2003" class="entry">
	<td>Heller MJ (2003), <i>"DNA Microarray Technology: Devices, Systems, and Applications"</i>, http://dx.doi.org/10.1146/annurev.bioeng.4.020702.153438., nov, 2003.  Vol. 4, pp. 129-153.  Annual Reviews 4139 El Camino Way, P.O. Box 10139, Palo Alto, CA 94303-0139, USA .
	<p class="infolinks">[<a href="javascript:toggleInfo('Heller2003','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Heller2003','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1146/ANNUREV.BIOENG.4.020702.153438" target="_blank">DOI</a>] [<a href="https://www.annualreviews.org/doi/abs/10.1146/annurev.bioeng.4.020702.153438" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Heller2003" class="abstract noshow">
	<td><b>Abstract</b>: ▪ Abstract In this review, recent advances in DNA microarray technology and their applications are examined. The many varieties of DNA microarray or DNA chip devices and systems are described along...</td>
</tr>
<tr id="bib_Heller2003" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Heller2003,
  author = {Heller, Michael J.},
  title = {DNA Microarray Technology: Devices, Systems, and Applications},
  journal = {http://dx.doi.org/10.1146/annurev.bioeng.4.020702.153438},
  publisher = { Annual Reviews 4139 El Camino Way, P.O. Box 10139, Palo Alto, CA 94303-0139, USA },
  year = {2003},
  volume = {4},
  pages = {129--153},
  url = {https://www.annualreviews.org/doi/abs/10.1146/annurev.bioeng.4.020702.153438},
  doi = {10.1146/ANNUREV.BIOENG.4.020702.153438}
}
</pre></td>
</tr>
<tr id="Hormozdiari2015" class="entry">
	<td>Hormozdiari F, Kichaev G, Yang WY, Pasaniuc B and Eskin E (2015), <i>"Identification of causal genes for complex traits"</i>, Bioinformatics., jun, 2015.  Vol. 31(12), pp. i206-i213. Oxford Academic.
	<p class="infolinks">[<a href="javascript:toggleInfo('Hormozdiari2015','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Hormozdiari2015','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1093/BIOINFORMATICS/BTV240" target="_blank">DOI</a>] [<a href="https://academic.oup.com/bioinformatics/article/31/12/i206/215786" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Hormozdiari2015" class="abstract noshow">
	<td><b>Abstract</b>: Motivation: Although genome-wide association studies (GWAS) have identified thousands of variants associated with common diseases and complex traits, only a handful of these variants are validated to be causal. We consider 'causal variants' as variants which are responsible for the association signal at a locus. As opposed to association studies that benefit from linkage disequilibrium (LD), the main challenge in identifying causal variants at associated loci lies in distinguishing among the many closely correlated variants due to LD. This is particularly important for model organisms such as inbred mice, where LD extends much further than in human populations, resulting in large stretches of the genome with significantly associated variants. Furthermore, these model organisms are highly structured and require correction for population structure to remove potential spurious associations. Results: In this work, we propose CAVIAR-Gene (CAusal Variants Identification in Associated Regions), a novel method that is able to operate across large LD regions of the genome while also correcting for population structure. A key feature of our approach is that it provides as output a minimally sized set of genes that captures the genes which harbor causal variants with probability &rho;. Through extensive simulations, we demonstrate that our method not only speeds up computation, but also have an average of 10% higher recall rate compared with the existing approaches. We validate our method using a real mouse high-density lipoprotein data (HDL) and show that CAVIAR-Gene is able to identify Apoa2 (a gene known to harbor causal variants for HDL), while reducing the number of genes that need to be tested for functionality by a factor of 2.</td>
</tr>
<tr id="bib_Hormozdiari2015" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Hormozdiari2015,
  author = {Hormozdiari, Farhad and Kichaev, Gleb and Yang, Wen Yun and Pasaniuc, Bogdan and Eskin, Eleazar},
  title = {Identification of causal genes for complex traits},
  journal = {Bioinformatics},
  publisher = {Oxford Academic},
  year = {2015},
  volume = {31},
  number = {12},
  pages = {i206--i213},
  url = {https://academic.oup.com/bioinformatics/article/31/12/i206/215786},
  doi = {10.1093/BIOINFORMATICS/BTV240}
}
</pre></td>
</tr>
<tr id="Ikegawa2012" class="entry">
	<td>Ikegawa S (2012), <i>"A Short History of the Genome-Wide Association Study: Where We Were and Where We Are Going"</i>, Genomics &amp; Informatics.  Vol. 10(4), pp. 220. Korea Genome Organization.
	<p class="infolinks">[<a href="javascript:toggleInfo('Ikegawa2012','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Ikegawa2012','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.5808/GI.2012.10.4.220" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC3543921/ /pmc/articles/PMC3543921/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3543921/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Ikegawa2012" class="abstract noshow">
	<td><b>Abstract</b>: Recent rapid advances in genetic research are ushering us into the genome sequence era, where an individual's genome information is utilized for clinical practice. The most spectacular results of the human genome study have been provided by genome-wide association studies (GWASs). This is a review of the history of GWASs as related to my work. Further efforts are necessary to make full use of its potential power to medicine.</td>
</tr>
<tr id="bib_Ikegawa2012" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Ikegawa2012,
  author = {Ikegawa, Shiro},
  title = {A Short History of the Genome-Wide Association Study: Where We Were and Where We Are Going},
  journal = {Genomics &amp; Informatics},
  publisher = {Korea Genome Organization},
  year = {2012},
  volume = {10},
  number = {4},
  pages = {220},
  url = {/pmc/articles/PMC3543921/ /pmc/articles/PMC3543921/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3543921/},
  doi = {10.5808/GI.2012.10.4.220}
}
</pre></td>
</tr>
<tr id="Isakov2017" class="entry">
	<td>Isakov O, Dotan I and Ben-Shachar S (2017), <i>"FUTURE DIRECTIONS AND METHODS FOR IBD RESEARCH Machine Learning-Based Gene Prioritization Identifies Novel Candidate Risk Genes for Inflammatory Bowel Disease Background: The inflammatory bowel diseases (IBDs) are chronic inflammatory disorders, associate"</i>, Computational and Mathematical Methods in Medicine. 
	<p class="infolinks"> [<a href="javascript:toggleInfo('Isakov2017','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1097/MIB.0000000000001222" target="_blank">DOI</a>] [<a href="www.ibdjournal.org" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="bib_Isakov2017" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Isakov2017,
  author = {Isakov, Ofer and Dotan, Iris and Ben-Shachar, Shay},
  title = {FUTURE DIRECTIONS AND METHODS FOR IBD RESEARCH Machine Learning-Based Gene Prioritization Identifies Novel Candidate Risk Genes for Inflammatory Bowel Disease Background: The inflammatory bowel diseases (IBDs) are chronic inflammatory disorders, associate},
  journal = {Computational and Mathematical Methods in Medicine},
  year = {2017},
  url = {www.ibdjournal.org},
  doi = {10.1097/MIB.0000000000001222}
}
</pre></td>
</tr>
<tr id="Julienne2021" class="entry">
	<td>Julienne H, Laville V, McCaw ZR, He Z, Guillemot V, Lasry C, Ziyatdinov A, Nerin C, Vaysse A, Lechat P, M&eacute;nager H, Le Goff W, Dube MP, Kraft P, Ionita-Laza I, Vilhj&aacute;lmsson BJ and Aschard H (2021), <i>"Multitrait GWAS to connect disease variants and biological mechanisms"</i>, PLOS Genetics., aug, 2021.  Vol. 17(8), pp. e1009713. Public Library of Science.
	<p class="infolinks">[<a href="javascript:toggleInfo('Julienne2021','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Julienne2021','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1371/JOURNAL.PGEN.1009713" target="_blank">DOI</a>] [<a href="https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1009713" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Julienne2021" class="abstract noshow">
	<td><b>Abstract</b>: Genome-wide association studies (GWASs) have uncovered a wealth of associations between common variants and human phenotypes. Here, we present an integrative analysis of GWAS summary statistics from 36 phenotypes to decipher multitrait genetic architecture and its link with biological mechanisms. Our framework incorporates multitrait association mapping along with an investigation of the breakdown of genetic associations into clusters of variants harboring similar multitrait association profiles. Focusing on two subsets of immunity and metabolism phenotypes, we then demonstrate how genetic variants within clusters can be mapped to biological pathways and disease mechanisms. Finally, for the metabolism set, we investigate the link between gene cluster assignment and the success of drug targets in randomized controlled trials.</td>
</tr>
<tr id="bib_Julienne2021" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Julienne2021,
  author = {Julienne, Hanna and Laville, Vincent and McCaw, Zachary R. and He, Zihuai and Guillemot, Vincent and Lasry, Carla and Ziyatdinov, Andrey and Nerin, Cyril and Vaysse, Amaury and Lechat, Pierre and M&eacute;nager, Herv&eacute; and Le Goff, Wilfried and Dube, Marie Pierre and Kraft, Peter and Ionita-Laza, Iuliana and Vilhj&aacute;lmsson, Bjarni J. and Aschard, Hugues},
  title = {Multitrait GWAS to connect disease variants and biological mechanisms},
  journal = {PLOS Genetics},
  publisher = {Public Library of Science},
  year = {2021},
  volume = {17},
  number = {8},
  pages = {e1009713},
  url = {https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1009713},
  doi = {10.1371/JOURNAL.PGEN.1009713}
}
</pre></td>
</tr>
<tr id="Kremer2017" class="entry">
	<td>Kremer LS, Bader DM, Mertes C, Kopajtich R, Pichler G, Iuso A, Haack TB, Graf E, Schwarzmayr T, Terrile C, Koňař&iacute;kova E, Repp B, Kastenm&uuml;ller G, Adamski J, Lichtner P, Leonhardt C, Funalot B, Donati A, Tiranti V, Lombes A, Jardel C, Gl&auml;ser D, Taylor RW, Ghezzi D, Mayr JA, R&ouml;tig A, Freisinger P, Distelmaier F, Strom TM, Meitinger T, Gagneur J and Prokisch H (2017), <i>"Genetic diagnosis of Mendelian disorders via RNA sequencing"</i>, Nature Communications 2017 8:1., jun, 2017.  Vol. 8(1), pp. 1-11. Nature Publishing Group.
	<p class="infolinks">[<a href="javascript:toggleInfo('Kremer2017','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Kremer2017','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1038/ncomms15824" target="_blank">DOI</a>] [<a href="https://www.nature.com/articles/ncomms15824" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Kremer2017" class="abstract noshow">
	<td><b>Abstract</b>: Across a variety of Mendelian disorders, ∼50–75% of patients do not receive a genetic diagnosis by exome sequencing indicating disease-causing variants in non-coding regions. Although genome sequencing in principle reveals all genetic variants, their sizeable number and poorer annotation make prioritization challenging. Here, we demonstrate the power of transcriptome sequencing to molecularly diagnose 10% (5 of 48) of mitochondriopathy patients and identify candidate genes for the remainder. We find a median of one aberrantly expressed gene, five aberrant splicing events and six mono-allelically expressed rare variants in patient-derived fibroblasts and establish disease-causing roles for each kind. Private exons often arise from cryptic splice sites providing an important clue for variant prioritization. One such event is found in the complex I assembly factor TIMMDC1 establishing a novel disease-associated gene. In conclusion, our study expands the diagnostic tools for detecting non-exonic variants and provides examples of intronic loss-of-function variants with pathological relevance. Genome sequencing alone fails to provide a genetic diagnosis for many Mendelian disorder patients. Here, the authors utilize RNA sequencing to complement genotyping of patients with a rare mitochondrial disease by detecting aberrant RNA expression, splicing and allele-specific expression.</td>
</tr>
<tr id="bib_Kremer2017" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Kremer2017,
  author = {Kremer, Laura S. and Bader, Daniel M. and Mertes, Christian and Kopajtich, Robert and Pichler, Garwin and Iuso, Arcangela and Haack, Tobias B. and Graf, Elisabeth and Schwarzmayr, Thomas and Terrile, Caterina and Koňař&iacute;kova, Eli&scaron;ka and Repp, Birgit and Kastenm&uuml;ller, Gabi and Adamski, Jerzy and Lichtner, Peter and Leonhardt, Christoph and Funalot, Benoit and Donati, Alice and Tiranti, Valeria and Lombes, Anne and Jardel, Claude and Gl&auml;ser, Dieter and Taylor, Robert W. and Ghezzi, Daniele and Mayr, Johannes A. and R&ouml;tig, Agnes and Freisinger, Peter and Distelmaier, Felix and Strom, Tim M. and Meitinger, Thomas and Gagneur, Julien and Prokisch, Holger},
  title = {Genetic diagnosis of Mendelian disorders via RNA sequencing},
  journal = {Nature Communications 2017 8:1},
  publisher = {Nature Publishing Group},
  year = {2017},
  volume = {8},
  number = {1},
  pages = {1--11},
  url = {https://www.nature.com/articles/ncomms15824},
  doi = {10.1038/ncomms15824}
}
</pre></td>
</tr>
<tr id="LaFramboise2009" class="entry">
	<td>LaFramboise T (2009), <i>"Single nucleotide polymorphism arrays: a decade of biological, computational and technological advances"</i>, Nucleic Acids Research.  Vol. 37(13), pp. 4181. Oxford University Press.
	<p class="infolinks">[<a href="javascript:toggleInfo('LaFramboise2009','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('LaFramboise2009','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1093/NAR/GKP552" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC2715261/ /pmc/articles/PMC2715261/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2715261/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_LaFramboise2009" class="abstract noshow">
	<td><b>Abstract</b>: Array manufacturers originally designed single nucleotide polymorphism (SNP) arrays to genotype human DNA at thousands of SNPs across the genome simultaneously. In the decade since their initial development, the platform's applications have expanded to include the detection and characterization of copy number variation-whether somatic, inherited, or de novo-as well as loss-of-heterozygosity in cancer cells. The technology's impressive contributions to insights in population and molecular genetics have been fueled by advances in computational methodology, and indeed these insights and methodologies have spurred developments in the arrays themselves. This review describes the most commonly used SNP array platforms, surveys the computational methodologies used to convert the raw data into inferences at the DNA level, and details the broad range of applications. Although the long-term future of SNP arrays is unclear, cost considerations ensure their relevance for at least the next several years. Even as emerging technologies seem poised to take over for at least some applications, researchers working with these new sources of data are adopting the computational approaches originally developed for SNP arrays.</td>
</tr>
<tr id="bib_LaFramboise2009" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{LaFramboise2009,
  author = {LaFramboise, Thomas},
  title = {Single nucleotide polymorphism arrays: a decade of biological, computational and technological advances},
  journal = {Nucleic Acids Research},
  publisher = {Oxford University Press},
  year = {2009},
  volume = {37},
  number = {13},
  pages = {4181},
  url = {/pmc/articles/PMC2715261/ /pmc/articles/PMC2715261/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2715261/},
  doi = {10.1093/NAR/GKP552}
}
</pre></td>
</tr>
<tr id="Le2020" class="entry">
	<td>Le DH (2020), <i>"Machine learning-based approaches for disease gene prediction"</i>, Briefings in Functional Genomics., dec, 2020.  Vol. 19(5-6), pp. 350-363. Oxford Academic.
	<p class="infolinks">[<a href="javascript:toggleInfo('Le2020','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Le2020','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1093/BFGP/ELAA013" target="_blank">DOI</a>] [<a href="https://academic.oup.com/bfg/article/19/5-6/350/5860123" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Le2020" class="abstract noshow">
	<td><b>Abstract</b>: Disease gene prediction is an essential issue in biomedical research. In the early days, annotation-based approaches were proposed for this problem. With the development of high-throughput technologies, interaction data between genes/proteins have grown quickly and covered almost genome and proteome; thus, network-based methods for the problem become prominent. In parallel, machine learning techniques, which formulate the problem as a classification, have also been proposed. Here, we firstly show a roadmap of the machine learning-based methods for the disease gene prediction. In the beginning, the problem was usually approached using a binary classification, where positive and negative training sample sets are comprised of disease genes and non-disease genes, respectively. The disease genes are ones known to be associated with diseases; meanwhile, non-disease genes were randomly selected from those not yet known to be associated with diseases. However, the later may contain unknown disease genes. To overcome this uncertainty of defining the non-disease genes, more realistic approaches have been proposed for the problem, such as unary and semi-supervised classification. Recently, more advanced methods, including ensemble learning, matrix factorization and deep learning, have been proposed for the problem. Secondly, 12 representative machine learning-based methods for the disease gene prediction were examined and compared in terms of prediction performance and running time. Finally, their advantages, disadvantages, interpretability and trust were also analyzed and discussed.</td>
</tr>
<tr id="bib_Le2020" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Le2020,
  author = {Le, Duc Hau},
  title = {Machine learning-based approaches for disease gene prediction},
  journal = {Briefings in Functional Genomics},
  publisher = {Oxford Academic},
  year = {2020},
  volume = {19},
  number = {5-6},
  pages = {350--363},
  url = {https://academic.oup.com/bfg/article/19/5-6/350/5860123},
  doi = {10.1093/BFGP/ELAA013}
}
</pre></td>
</tr>
<tr id="Lin2019" class="entry">
	<td>Lin F, Fan J and Rhee SY (2019), <i>"QTG-Finder: A Machine-Learning Based Algorithm To Prioritize Causal Genes of Quantitative Trait Loci in Arabidopsis and Rice"</i>, G3 Genes|Genomes|Genetics., oct, 2019.  Vol. 9(10), pp. 3129-3138. Oxford Academic.
	<p class="infolinks">[<a href="javascript:toggleInfo('Lin2019','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Lin2019','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1534/G3.119.400319" target="_blank">DOI</a>] [<a href="https://academic.oup.com/g3journal/article/9/10/3129/6026658" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Lin2019" class="abstract noshow">
	<td><b>Abstract</b>: Linkage mapping is one of the most commonly used methods to identify genetic loci that determine a trait. However, the loci identified by linkage mapping may contain hundreds of candidate genes and require a time-consuming and labor-intensive fine mapping process to find the causal gene controlling the trait. With the availability of a rich assortment of genomic and functional genomic data, it is possible to develop a computational method to facilitate faster identification of causal genes. We developed QTG-Finder, a machine learning based algorithm to prioritize causal genes by ranking genes within a quantitative trait locus (QTL). Two predictive models were trained separately based on known causal genes in Arabidopsis and rice. An independent validation analysis showed that the models could recall about 64% of Arabidopsis and 79% of rice causal genes when the top 20% ranked genes were considered. The top 20% ranked genes can range from 10 to 100 genes, depending on the size of a QTL. The models can prioritize different types of traits though at different efficiency. We also identified several important features of causal genes including paralog copy number, being a transporter, being a transcription factor, and containing SNPs that cause premature stop codon. This work lays the foundation for systematically understanding characteristics of causal genes and establishes a pipeline to predict causal genes based on public data.</td>
</tr>
<tr id="bib_Lin2019" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Lin2019,
  author = {Lin, Fan and Fan, Jue and Rhee, Seung Y.},
  title = {QTG-Finder: A Machine-Learning Based Algorithm To Prioritize Causal Genes of Quantitative Trait Loci in Arabidopsis and Rice},
  journal = {G3 Genes|Genomes|Genetics},
  publisher = {Oxford Academic},
  year = {2019},
  volume = {9},
  number = {10},
  pages = {3129--3138},
  url = {https://academic.oup.com/g3journal/article/9/10/3129/6026658},
  doi = {10.1534/G3.119.400319}
}
</pre></td>
</tr>
<tr id="Lin2020" class="entry">
	<td>Lin F, Lazarus EZ and Rhee SY (2020), <i>"QTG-Finder2: A Generalized Machine-Learning Algorithm for Prioritizing QTL Causal Genes in Plants"</i>, G3 Genes|Genomes|Genetics., jul, 2020.  Vol. 10(7), pp. 2411-2421. Oxford Academic.
	<p class="infolinks">[<a href="javascript:toggleInfo('Lin2020','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Lin2020','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1534/G3.120.401122" target="_blank">DOI</a>] [<a href="https://academic.oup.com/g3journal/article/10/7/2411/6026222" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Lin2020" class="abstract noshow">
	<td><b>Abstract</b>: Linkage mapping has been widely used to identify quantitative trait loci (QTL) in many plants and usually requires a time-consuming and labor-intensive fine mapping process to find the causal gene underlying the QTL. Previously, we described QTG-Finder, a machine-learning algorithm to rationally prioritize candidate causal genes in QTLs. While it showed good performance, QTG-Finder could only be used in Arabidopsis and rice because of the limited number of known causal genes in other species. Here we tested the feasibility of enabling QTG-Finder to work on species that have few or no known causal genes by using orthologs of known causal genes as the training set. The model trained with orthologs could recall about 64% of Arabidopsis and 83% of rice causal genes when the top 20% ranked genes were considered, which is similar to the performance of models trained with known causal genes. The average precision was 0.027 for Arabidopsis and 0.029 for rice. We further extended the algorithm to include polymorphisms in conserved non-coding sequences and gene presence/absence variation as additional features. Using this algorithm, QTG-Finder2, we trained and cross-validated Sorghum bicolor and Setaria viridis models. The S. bicolor model was validated by causal genes curated from the literature and could recall 70% of causal genes when the top 20% ranked genes were considered. In addition, we applied the S. viridis model and public transcriptome data to prioritize a plant height QTL and identified 13 candidate genes. QTL-Finder2 can accelerate the discovery of causal genes in any plant species and facilitate agricultural trait improvement.</td>
</tr>
<tr id="bib_Lin2020" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Lin2020,
  author = {Lin, Fan and Lazarus, Elena Z. and Rhee, Seung Y.},
  title = {QTG-Finder2: A Generalized Machine-Learning Algorithm for Prioritizing QTL Causal Genes in Plants},
  journal = {G3 Genes|Genomes|Genetics},
  publisher = {Oxford Academic},
  year = {2020},
  volume = {10},
  number = {7},
  pages = {2411--2421},
  url = {https://academic.oup.com/g3journal/article/10/7/2411/6026222},
  doi = {10.1534/G3.120.401122}
}
</pre></td>
</tr>
<tr id="Mccarthy1995" class="entry">
	<td>Mccarthy L, Huntert K, Schalkwyk L, Ribat L, Anson S, Morr R, Newell W, Bruley C, Bart I, Ramut E, Housmant D, Cox R and Lehrach H (1995), <i>"Efficient high-resolution genetic mapping of mouse interspersed repetitive sequence PCR products, toward integrated genetic and physical mapping of the mouse genome"</i>, Genetics.  Vol. 92, pp. 5302-5306.
	<p class="infolinks">[<a href="javascript:toggleInfo('Mccarthy1995','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Mccarthy1995','bibtex')">BibTeX</a>] [<a href="http://gea.lif.icnet." target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Mccarthy1995" class="abstract noshow">
	<td><b>Abstract</b>: The ability to carry out high-resolution genetic mapping at high throughput in the mouse is a critical rate-limiting step in the generation of genetically anchored contigs in physical mapping projects and the mapping of genetic loci for complex traits. To address this need, we have developed an efficient, high-resolution, large-scale genome mapping system. This system is based on the identification of polymorphic DNA sites between mouse strains by using interspersed repetitive sequence (IRS) PCR. Individual cloned IRS PCR products are hybridized to a DNA array of IRS PCR products derived from the DNA of individual mice segregating DNA sequences from the two parent strains. Since gel elec-trophoresis is not required, large numbers of samples can be genotyped in parallel. By using this approach, we have mapped >450 polymorphic probes with filters containing the DNA of up to 517 backcross mice, potentially allowing resolution of 0.14 centimorgan. This approach also carries the potential for a high degree of efficiency in the integration of physical and genetic maps, since pooled DNAs representing libraries of yeast artificial chromosomes or other physical representations of the mouse genome can be addressed by hybridization of filter representations of the IRS PCR products of such libraries. Mouse genetics is one of the most powerful tools for the analysis of almost all aspects of vertebrate biology including development, physiology, and pathobiology. Mouse models have already been developed for many human diseases (1-6).</td>
</tr>
<tr id="bib_Mccarthy1995" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Mccarthy1995,
  author = {Mccarthy, Linda and Huntert, Kenr and Schalkwyk, Leonard and Ribat, Laura and Anson, Simon and Morr, Richard and Newell, William and Bruley, Charlotre and Bart, Isabelle and Ramut, Elango and Housmant, David and Cox, Roger and Lehrach, Hans},
  title = {Efficient high-resolution genetic mapping of mouse interspersed repetitive sequence PCR products, toward integrated genetic and physical mapping of the mouse genome},
  journal = {Genetics},
  year = {1995},
  volume = {92},
  pages = {5302--5306},
  url = {http://gea.lif.icnet.}
}
</pre></td>
</tr>
<tr id="Mordelet2011" class="entry">
	<td>Mordelet F and Vert JP (2011), <i>"ProDiGe: Prioritization Of Disease Genes with multitask machine learning from positive and unlabeled examples"</i>, BMC Bioinformatics., oct, 2011.  Vol. 12(1), pp. 1-15. BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('Mordelet2011','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Mordelet2011','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/1471-2105-12-389/TABLES/6" target="_blank">DOI</a>] [<a href="https://link.springer.com/articles/10.1186/1471-2105-12-389 https://link.springer.com/article/10.1186/1471-2105-12-389" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Mordelet2011" class="abstract noshow">
	<td><b>Abstract</b>: Background: Elucidating the genetic basis of human diseases is a central goal of genetics and molecular biology. While traditional linkage analysis and modern high-throughput techniques often provide long lists of tens or hundreds of disease gene candidates, the identification of disease genes among the candidates remains time-consuming and expensive. Efficient computational methods are therefore needed to prioritize genes within the list of candidates, by exploiting the wealth of information available about the genes in various databases.Results: We propose ProDiGe, a novel algorithm for Prioritization of Disease Genes. ProDiGe implements a novel machine learning strategy based on learning from positive and unlabeled examples, which allows to integrate various sources of information about the genes, to share information about known disease genes across diseases, and to perform genome-wide searches for new disease genes. Experiments on real data show that ProDiGe outperforms state-of-the-art methods for the prioritization of genes in human diseases.Conclusions: ProDiGe implements a new machine learning paradigm for gene prioritization, which could help the identification of new disease genes. It is freely available at http://cbio.ensmp.fr/prodige. textcopyright 2011 Mordelet and Vert; licensee BioMed Central Ltd.</td>
</tr>
<tr id="bib_Mordelet2011" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Mordelet2011,
  author = {Mordelet, Fantine and Vert, Jean Philippe},
  title = {ProDiGe: Prioritization Of Disease Genes with multitask machine learning from positive and unlabeled examples},
  journal = {BMC Bioinformatics},
  publisher = {BioMed Central},
  year = {2011},
  volume = {12},
  number = {1},
  pages = {1--15},
  url = {https://link.springer.com/articles/10.1186/1471-2105-12-389 https://link.springer.com/article/10.1186/1471-2105-12-389},
  doi = {10.1186/1471-2105-12-389/TABLES/6}
}
</pre></td>
</tr>
<tr id="Murray1994" class="entry">
	<td>Murray JC, Buetow KH, Weber JL, Ludwigsen S, Scherpbier-Heddema T, Manion F, Quillen J, Sheffield VC, Sunden S, Duyk GM, Weissenbach J, Gyapay G, Dib C, Morrissette J, Mark Lathrop G, Vignal A, Matsunami N, Gerken S, Melis R, Albertsen H, Plaetke R, Odelberg S, Dausset J, Cohen D, Cann H, Buetow KH, Ludwigsen S, Scherpbier-Heddema T, Manion F, Quillen J, Weissenbach J, Gyapay G, Dib C, Vignal A, White R, Matsunami N, Gerken S, Melis R, Albertsen H, Plaetke R, Dausset J, Cohen D and Cann H (1994), <i>"A Comprehensive Human Linkage Map with Centimorgan Density Cooperative Human Linkage Center (CHLC)"</i>, Science. 
	<p class="infolinks">[<a href="javascript:toggleInfo('Murray1994','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Murray1994','bibtex')">BibTeX</a>] [<a href="https://www.science.org" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Murray1994" class="abstract noshow">
	<td><b>Abstract</b>: In the last few years there have been rapid advances in developing genetic maps for humans, greatly enhancing our ability to localize and identify genes for inherited disorders. Through the collaborative efforts of three large groups generating microsatellite markers and the efforts of the 1 10 CEPH collaborators, a comprehensive human linkage map is presented here. It consists of 5840 loci, of which 970 are uniquely ordered, covering 4000 centimorgans on the sex-averaged map. Of these loci, 3617 are polymerase chain reaction formatted short tandem repeat polymorphisms, and another 427 are genes. The map has markers at an average density of 0.7 centimorgan, providing a resource for ready transference to physical maps and achieving one of the first goals of the Human Genome Project-a comprehensive, high-density genetic map. For the first time, humans have been presented with the capability of understanding their own genetic makeup and how it contributes to morbidity of the individual and the species. Rapid scientific advances have made this possible, and developments in molecular biology, genetics , and computing, coupled with a cooperative and interactive biomedical community , have accelerated the progress of investigation into human inherited disorders. A primary engine driving these advances has been the development and use of human J. C. Murray is in the Departments of Pediatrics and</td>
</tr>
<tr id="bib_Murray1994" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Murray1994,
  author = {Murray, Jeffrey C and Buetow, Kenneth H and Weber, James L and Ludwigsen, Susan and Scherpbier-Heddema, Titia and Manion, Frank and Quillen, John and Sheffield, Val C and Sunden, Sara and Duyk, Geoffrey M and Weissenbach, Jean and Gyapay, Gabor and Dib, Colette and Morrissette, Jean and Mark Lathrop, G and Vignal, Alain and Matsunami, Norisada and Gerken, Steven and Melis, Roberta and Albertsen, Hans and Plaetke, Rosemarie and Odelberg, Shannon and Dausset, Jean and Cohen, Daniel and Cann, Howard and Buetow, K H and Ludwigsen, S and Scherpbier-Heddema, T and Manion, F and Quillen, J and Weissenbach, J and Gyapay, G and Dib, C and Vignal, A and White, R and Matsunami, N and Gerken, S and Melis, R and Albertsen, H and Plaetke, R and Dausset, J and Cohen, D and Cann, H},
  title = {A Comprehensive Human Linkage Map with Centimorgan Density Cooperative Human Linkage Center (CHLC)},
  journal = {Science},
  year = {1994},
  url = {https://www.science.org}
}
</pre></td>
</tr>
<tr id="Nelson2011" class="entry">
	<td>Nelson FK, Snyder M, Gardner AF, Hendrickson CL, Shendure JA, Porreca GJ, Church GM, Ausubel FM, Ju J, Kieleczawa J and Slatko BE (2011), <i>"Introduction and historical overview of DNA sequencing"</i>, Current Protocols in Molecular Biology. (SUPPL.96), pp. 1-18.
	<p class="infolinks">[<a href="javascript:toggleInfo('Nelson2011','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Nelson2011','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1002/0471142727.mb0700s96" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Nelson2011" class="abstract noshow">
	<td><b>Abstract</b>: The process of DNA sequencing has made tremendous strides in throughput, improved accuracy, ease of production, and lowered cost. As the practice of DNA sequencing has improved, so has the downstream data analysis with sophisticated databases and bioinformatics tools. Together, these advances have enlarged the number of applications upon which DNA sequencing can be brought to bear. This introductory unit provides a description of DNA sequencing with a focus on current and "NextGen" (second and third generation) automated technologies and applications. textcopyright 2011 by John Wiley &amp; Sons, Inc.</td>
</tr>
<tr id="bib_Nelson2011" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Nelson2011,
  author = {Nelson, F. Kenneth and Snyder, Michael and Gardner, Andrew F. and Hendrickson, Cynthia L. and Shendure, Jay A. and Porreca, Gregory J. and Church, George M. and Ausubel, Frederick M. and Ju, Jingyue and Kieleczawa, Jan and Slatko, Barton E.},
  title = {Introduction and historical overview of DNA sequencing},
  journal = {Current Protocols in Molecular Biology},
  year = {2011},
  number = {SUPPL.96},
  pages = {1--18},
  doi = {10.1002/0471142727.mb0700s96}
}
</pre></td>
</tr>
<tr id="Nicholls2020" class="entry">
	<td>Nicholls HL, John CR, Watson DS, Munroe PB, Barnes MR and Cabrera CP (2020), <i>"Reaching the End-Game for GWAS: Machine Learning Approaches for the Prioritization of Complex Disease Loci"</i>, Frontiers in Genetics., apr, 2020.  Vol. 11, pp. 350. Frontiers Media S.A..
	<p class="infolinks">[<a href="javascript:toggleInfo('Nicholls2020','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Nicholls2020','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.3389/FGENE.2020.00350/BIBTEX" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Nicholls2020" class="abstract noshow">
	<td><b>Abstract</b>: Genome-wide association studies (GWAS) have revealed thousands of genetic loci that underpin the complex biology of many human traits. However, the strength of GWAS – the ability to detect genetic association by linkage disequilibrium (LD) – is also its limitation. Whilst the ever-increasing study size and improved design have augmented the power of GWAS to detect effects, differentiation of causal variants or genes from other highly correlated genes associated by LD remains the real challenge. This has severely hindered the biological insights and clinical translation of GWAS findings. Although thousands of disease susceptibility loci have been reported, causal genes at these loci remain elusive. Machine learning (ML) techniques offer an opportunity to dissect the heterogeneity of variant and gene signals in the post-GWAS analysis phase. ML models for GWAS prioritization vary greatly in their complexity, ranging from relatively simple logistic regression approaches to more complex ensemble models such as random forests and gradient boosting, as well as deep learning models, i.e., neural networks. Paired with functional validation, these methods show important promise for clinical translation, providing a strong evidence-based approach to direct post-GWAS research. However, as ML approaches continue to evolve to meet the challenge of causal gene identification, a critical assessment of the underlying methodologies and their applicability to the GWAS prioritization problem is needed. This review investigates the landscape of ML applications in three parts: selected models, input features, and output model performance, with a focus on prioritizations of complex disease associated loci. Overall, we explore the contributions ML has made towards reaching the GWAS end-game with consequent wide-ranging translational impact.</td>
</tr>
<tr id="bib_Nicholls2020" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Nicholls2020,
  author = {Nicholls, Hannah L. and John, Christopher R. and Watson, David S. and Munroe, Patricia B. and Barnes, Michael R. and Cabrera, Claudia P.},
  title = {Reaching the End-Game for GWAS: Machine Learning Approaches for the Prioritization of Complex Disease Loci},
  journal = {Frontiers in Genetics},
  publisher = {Frontiers Media S.A.},
  year = {2020},
  volume = {11},
  pages = {350},
  doi = {10.3389/FGENE.2020.00350/BIBTEX}
}
</pre></td>
</tr>
<tr id="Nitsch2010" class="entry">
	<td>Nitsch D, Gon&ccedil;alves JP, Ojeda F, de Moor B and Moreau Y (2010), <i>"Candidate gene prioritization by network analysis of differential expression using machine learning approaches"</i>, BMC Bioinformatics., sep, 2010.  Vol. 11(1), pp. 1-16. BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('Nitsch2010','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Nitsch2010','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/1471-2105-11-460/TABLES/3" target="_blank">DOI</a>] [<a href="https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-460" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Nitsch2010" class="abstract noshow">
	<td><b>Abstract</b>: Background: Discovering novel disease genes is still challenging for diseases for which no prior knowledge - such as known disease genes or disease-related pathways - is available. Performing genetic studies frequently results in large lists of candidate genes of which only few can be followed up for further investigation. We have recently developed a computational method for constitutional genetic disorders that identifies the most promising candidate genes by replacing prior knowledge by experimental data of differential gene expression between affected and healthy individuals.To improve the performance of our prioritization strategy, we have extended our previous work by applying different machine learning approaches that identify promising candidate genes by determining whether a gene is surrounded by highly differentially expressed genes in a functional association or protein-protein interaction network.Results: We have proposed three strategies scoring disease candidate genes relying on network-based machine learning approaches, such as kernel ridge regression, heat kernel, and Arnoldi kernel approximation. For comparison purposes, a local measure based on the expression of the direct neighbors is also computed. We have benchmarked these strategies on 40 publicly available knockout experiments in mice, and performance was assessed against results obtained using a standard procedure in genetics that ranks candidate genes based solely on their differential expression levels (Simple Expression Ranking). Our results showed that our four strategies could outperform this standard procedure and that the best results were obtained using the Heat Kernel Diffusion Ranking leading to an average ranking position of 8 out of 100 genes, an AUC value of 92.3% and an error reduction of 52.8% relative to the standard procedure approach which ranked the knockout gene on average at position 17 with an AUC value of 83.7%.Conclusion: In this study we could identify promising candidate genes using network based machine learning approaches even if no knowledge is available about the disease or phenotype. textcopyright 2010 Nitsch et al; licensee BioMed Central Ltd.</td>
</tr>
<tr id="bib_Nitsch2010" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Nitsch2010,
  author = {Nitsch, Daniela and Gon&ccedil;alves, Joana P. and Ojeda, Fabian and de Moor, Bart and Moreau, Yves},
  title = {Candidate gene prioritization by network analysis of differential expression using machine learning approaches},
  journal = {BMC Bioinformatics},
  publisher = {BioMed Central},
  year = {2010},
  volume = {11},
  number = {1},
  pages = {1--16},
  url = {https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-460},
  doi = {10.1186/1471-2105-11-460/TABLES/3}
}
</pre></td>
</tr>
<tr id="Nuel2017" class="entry">
	<td>Nuel G, Lefebvre A and Bouaziz O (2017), <i>"Computing Individual Risks Based on Family History in Genetic Disease in the Presence of Competing Risks"</i>, Computational and Mathematical Methods in Medicine.  Vol. 2017 Hindawi Limited.
	<p class="infolinks">[<a href="javascript:toggleInfo('Nuel2017','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Nuel2017','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1155/2017/9193630" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Nuel2017" class="abstract noshow">
	<td><b>Abstract</b>: When considering a genetic disease with variable age at onset (e.g., familial amyloid neuropathy, cancers), computing the individual risk of the disease based on family history (FH) is of critical interest for both clinicians and patients. Such a risk is very challenging to compute because 1 the genotype X of the individual of interest is in general unknown, 2 the posterior distribution PX|FH,T>t changes with t (T is the age at disease onset for the targeted individual), and 3 the competing risk of death is not negligible. In this work, we present modeling of this problem using a Bayesian network mixed with (right-censored) survival outcomes where hazard rates only depend on the genotype of each individual. We explain how belief propagation can be used to obtain posterior distribution of genotypes given the FH and how to obtain a time-dependent posterior hazard rate for any individual in the pedigree. Finally, we use this posterior hazard rate to compute individual risk, with or without the competing risk of death. Our method is illustrated using the Claus-Easton model for breast cancer. The competing risk of death is derived from the national French registry.</td>
</tr>
<tr id="bib_Nuel2017" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Nuel2017,
  author = {Nuel, Gregory and Lefebvre, Alexandra and Bouaziz, Olivier},
  title = {Computing Individual Risks Based on Family History in Genetic Disease in the Presence of Competing Risks},
  journal = {Computational and Mathematical Methods in Medicine},
  publisher = {Hindawi Limited},
  year = {2017},
  volume = {2017},
  doi = {10.1155/2017/9193630}
}
</pre></td>
</tr>
<tr id="Ohtsuka2006" class="entry">
	<td>Ohtsuka Y, Wang XT, Saito J, Ishida T and Munakata M (2006), <i>"Genetic linkage analysis of pulmonary fibrotic response to silica in mice"</i>, European Respiratory Journal., nov, 2006.  Vol. 28(5), pp. 1013-1019. European Respiratory Society.
	<p class="infolinks">[<a href="javascript:toggleInfo('Ohtsuka2006','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Ohtsuka2006','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1183/09031936.06.00132505" target="_blank">DOI</a>] [<a href="https://erj.ersjournals.com/content/28/5/1013 https://erj.ersjournals.com/content/28/5/1013.abstract" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Ohtsuka2006" class="abstract noshow">
	<td><b>Abstract</b>: Inter-individual variations in the development of silicosis, even within the same environments, have been reported, which suggest the contribution of genetic factors in silicosis aetiology. The aim of the present study was to determine whether there is any significant genetic influence on the development of silicosis. Furthermore, which genetic loci are responsible for the pulmonary response to silica exposure?<p>Eight strains of inbred mice were used to examine the genetic influence on the lung fibrotic response to silica exposure. After intercross-breeding between the most susceptible and most resistant strains, a genome-wide linkage analysis of quantitative trait loci (QTL) was performed. Hydroxyproline was applied as an index, and genotypes of 167 marker genes were analysed by fragment analysis using a capillary-type sequencer.<p>There was significant inter-strain difference in the mean concentration of hydroxyproline contents among the eight strains of mice. Breeding studies were conducted between the most susceptible, C57BL/6J, and the most resistant strain, CBA/J. A genome-wide linkage analysis of silica-exposed intercrossed cohorts identified significant QTL on chromosome 4 and suggestive QTL on chromosomes 3 and 18.<p>The present study demonstrates that genetic factors may play a significant role in fibrotic-lung responses to silica; one significant and two suggestive quantitative trait loci were identified.</td>
</tr>
<tr id="bib_Ohtsuka2006" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Ohtsuka2006,
  author = {Ohtsuka, Y. and Wang, X. T. and Saito, J. and Ishida, T. and Munakata, M.},
  title = {Genetic linkage analysis of pulmonary fibrotic response to silica in mice},
  journal = {European Respiratory Journal},
  publisher = {European Respiratory Society},
  year = {2006},
  volume = {28},
  number = {5},
  pages = {1013--1019},
  url = {https://erj.ersjournals.com/content/28/5/1013 https://erj.ersjournals.com/content/28/5/1013.abstract},
  doi = {10.1183/09031936.06.00132505}
}
</pre></td>
</tr>
<tr id="Pareek2011" class="entry">
	<td>Pareek CS, Smoczynski R and Tretyn A (2011), <i>"Sequencing technologies and genome sequencing"</i>, Journal of Applied Genetics 2011 52:4., jun, 2011.  Vol. 52(4), pp. 413-435. Springer.
	<p class="infolinks">[<a href="javascript:toggleInfo('Pareek2011','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Pareek2011','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1007/S13353-011-0057-X" target="_blank">DOI</a>] [<a href="https://link.springer.com/article/10.1007/s13353-011-0057-x" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Pareek2011" class="abstract noshow">
	<td><b>Abstract</b>: The high-throughput - next generation sequencing (HT-NGS) technologies are currently the hottest topic in the field of human and animals genomics researches, which can produce over 100 times more data compared to the most sophisticated capillary sequencers based on the Sanger method. With the ongoing developments of high throughput sequencing machines and advancement of modern bioinformatics tools at unprecedented pace, the target goal of sequencing individual genomes of living organism at a cost of $1,000 each is seemed to be realistically feasible in the near future. In the relatively short time frame since 2005, the HT-NGS technologies are revolutionizing the human and animal genome researches by analysis of chromatin immunoprecipitation coupled to DNA microarray (ChIP-chip) or sequencing (ChIP-seq), RNA sequencing (RNA-seq), whole genome genotyping, genome wide structural variation, de novo assembling and re-assembling of genome, mutation detection and carrier screening, detection of inherited disorders and complex human diseases, DNA library preparation, paired ends and genomic captures, sequencing of mitochondrial genome and personal genomics. In this review, we addressed the important features of HT-NGS like, first generation DNA sequencers, birth of HT-NGS, second generation HT-NGS platforms, third generation HT-NGS platforms: including single molecule Heliscope™, SMRT™ and RNAP sequencers, Nanopore, Archon Genomics X PRIZE foundation, comparison of second and third HT-NGS platforms, applications, advances and future perspectives of sequencing technologies on human and animal genome research.</td>
</tr>
<tr id="bib_Pareek2011" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Pareek2011,
  author = {Pareek, Chandra Shekhar and Smoczynski, Rafal and Tretyn, Andrzej},
  title = {Sequencing technologies and genome sequencing},
  journal = {Journal of Applied Genetics 2011 52:4},
  publisher = {Springer},
  year = {2011},
  volume = {52},
  number = {4},
  pages = {413--435},
  url = {https://link.springer.com/article/10.1007/s13353-011-0057-x},
  doi = {10.1007/S13353-011-0057-X}
}
</pre></td>
</tr>
<tr id="Petersen2017" class="entry">
	<td>Petersen BS, Fredrich B, Hoeppner MP, Ellinghaus D and Franke A (2017), <i>"Opportunities and challenges of whole-genome and -exome sequencing"</i>, BMC Genetics., feb, 2017.  Vol. 18(1) BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('Petersen2017','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Petersen2017','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/S12863-017-0479-5" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC5307692/ /pmc/articles/PMC5307692/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5307692/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Petersen2017" class="abstract noshow">
	<td><b>Abstract</b>: Recent advances in the development of sequencing technologies provide researchers with unprecedented possibilities for genetic analyses. In this review, we will discuss the history of genetic studies and the progress driven by next-generation sequencing (NGS), using complex inflammatory bowel diseases as an example. We focus on the opportunities, but also challenges that researchers are facing when working with NGS data to unravel the genetic causes underlying diseases.</td>
</tr>
<tr id="bib_Petersen2017" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Petersen2017,
  author = {Petersen, Britt Sabina and Fredrich, Broder and Hoeppner, Marc P. and Ellinghaus, David and Franke, Andre},
  title = {Opportunities and challenges of whole-genome and -exome sequencing},
  journal = {BMC Genetics},
  publisher = {BioMed Central},
  year = {2017},
  volume = {18},
  number = {1},
  url = {/pmc/articles/PMC5307692/ /pmc/articles/PMC5307692/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5307692/},
  doi = {10.1186/S12863-017-0479-5}
}
</pre></td>
</tr>
<tr id="Raj2018" class="entry">
	<td>Raj RM and Sreeja A (2018), <i>"Analysis of Computational Gene Prioritization Approaches"</i>, Procedia Computer Science., jan, 2018.  Vol. 143, pp. 395-410. Elsevier.
	<p class="infolinks">[<a href="javascript:toggleInfo('Raj2018','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Raj2018','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1016/J.PROCS.2018.10.411" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_Raj2018" class="abstract noshow">
	<td><b>Abstract</b>: Even though biological data analysis helps in understanding the chemical processes, handling them is difficult due to their abundant size, heterogeneous nature and access time overheads. Complex disorder diagnostics can be carried out effectively by recognizing the most conformant genes from a set of candidate genes which are having a higher association with the disorder. Traditional gene analysis methods such as gene mutation analysis, single nucleotide polymorphism (SNP) detection and other wet lab techniques are delimited by several factors such as high-cost clinical experiments, unpredictable time consumption and insufficient prior knowledge about genetic materials. They are replaced by efficient computational solutions due to extensive advantages like economical computational cost, appropriate testing and validation strategies, adequate prior information etc. This paper contains a thorough literature review about prevailing methods, tools and data sources primarily used for computational gene prioritization. The aggregation, analysis, interpretation and comparison of different gene prioritization strategies are done with a view to provide an insight into recent trends and traits persist in them. Different validation methods commonly used for gene prioritization are also analysed in this study.</td>
</tr>
<tr id="bib_Raj2018" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Raj2018,
  author = {Raj, Rahul M. and Sreeja, A.},
  title = {Analysis of Computational Gene Prioritization Approaches},
  journal = {Procedia Computer Science},
  publisher = {Elsevier},
  year = {2018},
  volume = {143},
  pages = {395--410},
  doi = {10.1016/J.PROCS.2018.10.411}
}
</pre></td>
</tr>
<tr id="Rao2018" class="entry">
	<td>Rao A, Vg S, Joseph T, Kotte S, Sivadasan N and Srinivasan R (2018), <i>"Phenotype-driven gene prioritization for rare diseases using graph convolution on heterogeneous networks"</i>, BMC Medical Genomics., jul, 2018.  Vol. 11(1), pp. 1-12. BioMed Central Ltd..
	<p class="infolinks">[<a href="javascript:toggleInfo('Rao2018','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Rao2018','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/S12920-018-0372-8/FIGURES/7" target="_blank">DOI</a>] [<a href="https://bmcmedgenomics.biomedcentral.com/articles/10.1186/s12920-018-0372-8" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Rao2018" class="abstract noshow">
	<td><b>Abstract</b>: Background: One of the major goals of genomic medicine is the identification of causal genomic variants in a patient and their relation to the observed clinical phenotypes. Prioritizing the genomic variants by considering only the genotype information usually identifies a few hundred potential variants. Narrowing it down further to find the causal disease genes and relating them to the observed clinical phenotypes remains a significant challenge, especially for rare diseases. Methods: We propose a phenotype-driven gene prioritization approach using heterogeneous networks in the context of rare diseases. Towards this, we first built a heterogeneous network consisting of ontological associations as well as curated associations involving genes, diseases, phenotypes and pathways from multiple sources. Motivated by the recent progress in spectral graph convolutions, we developed a graph convolution based technique to infer new phenotype-gene associations from this initial set of associations. We included these inferred associations in the initial network and termed this integrated network HANRD (Heterogeneous Association Network for Rare Diseases). We validated this approach on 230 recently published rare disease clinical cases using the case phenotypes as input. Results: When HANRD was queried with the case phenotypes as input, the causal genes were captured within Top-50 for more than 31% of the cases and within Top-200 for more than 56% of the cases. The results showed improved performance when compared to other state-of-the-art tools. Conclusions: In this study, we showed that the heterogeneous network HANRD, consisting of curated, ontological and inferred associations, helped improve causal gene identification in rare diseases. HANRD allows future enhancements by supporting incorporation of new entity types and additional information sources.</td>
</tr>
<tr id="bib_Rao2018" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Rao2018,
  author = {Rao, Aditya and Vg, Saipradeep and Joseph, Thomas and Kotte, Sujatha and Sivadasan, Naveen and Srinivasan, Rajgopal},
  title = {Phenotype-driven gene prioritization for rare diseases using graph convolution on heterogeneous networks},
  journal = {BMC Medical Genomics},
  publisher = {BioMed Central Ltd.},
  year = {2018},
  volume = {11},
  number = {1},
  pages = {1--12},
  url = {https://bmcmedgenomics.biomedcentral.com/articles/10.1186/s12920-018-0372-8},
  doi = {10.1186/S12920-018-0372-8/FIGURES/7}
}
</pre></td>
</tr>
<tr id="Rao2021" class="entry">
	<td>Rao S, Yao Y and Bauer DE (2021), <i>"Editing GWAS: experimental approaches to dissect and exploit disease-associated genetic variation"</i>, Genome Medicine 2021 13:1., mar, 2021.  Vol. 13(1), pp. 1-20. BioMed Central.
	<p class="infolinks">[<a href="javascript:toggleInfo('Rao2021','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Rao2021','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1186/S13073-021-00857-3" target="_blank">DOI</a>] [<a href="https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-021-00857-3" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Rao2021" class="abstract noshow">
	<td><b>Abstract</b>: Genome-wide association studies (GWAS) have uncovered thousands of genetic variants that influence risk for human diseases and traits. Yet understanding the mechanisms by which these genetic variants, mainly noncoding, have an impact on associated diseases and traits remains a significant hurdle. In this review, we discuss emerging experimental approaches that are being applied for functional studies of causal variants and translational advances from GWAS findings to disease prevention and treatment. We highlight the use of genome editing technologies in GWAS functional studies to modify genomic sequences, with proof-of-principle examples. We discuss the challenges in interrogating causal variants, points for consideration in experimental design and interpretation of GWAS locus mechanisms, and the potential for novel therapeutic opportunities. With the accumulation of knowledge of functional genetics, therapeutic genome editing based on GWAS discoveries will become increasingly feasible.</td>
</tr>
<tr id="bib_Rao2021" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Rao2021,
  author = {Rao, Shuquan and Yao, Yao and Bauer, Daniel E.},
  title = {Editing GWAS: experimental approaches to dissect and exploit disease-associated genetic variation},
  journal = {Genome Medicine 2021 13:1},
  publisher = {BioMed Central},
  year = {2021},
  volume = {13},
  number = {1},
  pages = {1--20},
  url = {https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-021-00857-3},
  doi = {10.1186/S13073-021-00857-3}
}
</pre></td>
</tr>
<tr id="Sander1976" class="entry">
	<td>Sander F and Goulson AR (1976), <i>"A Rapid Method for Determining Sequences in DNA by Primed Synthesis with DNA Polymerase"</i>, J. Mol. Bid.  Vol. 94, pp. 441-448.
	<p class="infolinks">[<a href="javascript:toggleInfo('Sander1976','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Sander1976','bibtex')">BibTeX</a>]</p>
	</td>
</tr>
<tr id="abs_Sander1976" class="abstract noshow">
	<td><b>Abstract</b>: A simple and rapid method for d&amp;e rmining nucleotide sequences in single-stranded DNA by primed synthesis with DNA polymerace is described. It depends on the use of Ewherichia wli DNA polymemse I end DNA polymerase from bacteriophage T4 under conditions of different limiting nucleoside triphoaphates and concurrent fractionation of the products according to size by ionophoresis on acrylamide gels. The method was used to determine two sequences in bacterio-phage $X174 DNA using the synthetic decanucleotide A</td>
</tr>
<tr id="bib_Sander1976" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Sander1976,
  author = {Sander, F and Goulson, A R},
  title = {A Rapid Method for Determining Sequences in DNA by Primed Synthesis with DNA Polymerase},
  journal = {J. Mol. Bid},
  year = {1976},
  volume = {94},
  pages = {441--448}
}
</pre></td>
</tr>
<tr id="Sanger1973" class="entry">
	<td>Sanger F, Donelson JE, Coulson AR, K&ouml;ssel H and Fischer D (1973), <i>"Use of DNA Polymerase I Primed by a Synthetic Oligonucleotide to Determine a Nucleotide Sequence in Phage f1 DNA"</i>, Proceedings of the National Academy of Sciences of the United States of America.  Vol. 70(4), pp. 1209. National Academy of Sciences.
	<p class="infolinks">[<a href="javascript:toggleInfo('Sanger1973','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Sanger1973','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1073/PNAS.70.4.1209" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC433459/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC433459/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Sanger1973" class="abstract noshow">
	<td><b>Abstract</b>: A sequence of 50 residues in fl DNA has been determined by the extension of a chemically synthesized octadeoxyribonucleotide by Escherichia coli DNA polymerase I, with radioactive nucleoside triphos-phates and fl DNA template. The polymerized product was synthesized either in the presence of manganese and a mixture of ribo-and deoxyribotriphosphates or in a magnesium containing reaction with one or more of the four triphosphates absent. The sequence determination depended largely on fractionation of the polymerized products by two-dimensional "homochromatography." This approach and the techniques for the subsequent sequence analysis should be of general use for determining other sequences of DNA. Several features of this sequence suggest that it is located in an intercistronic region of fl DNA.</td>
</tr>
<tr id="bib_Sanger1973" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Sanger1973,
  author = {Sanger, F. and Donelson, J. E. and Coulson, A. R. and K&ouml;ssel, H. and Fischer, D.},
  title = {Use of DNA Polymerase I Primed by a Synthetic Oligonucleotide to Determine a Nucleotide Sequence in Phage f1 DNA},
  journal = {Proceedings of the National Academy of Sciences of the United States of America},
  publisher = {National Academy of Sciences},
  year = {1973},
  volume = {70},
  number = {4},
  pages = {1209},
  url = {/pmc/articles/PMC433459/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC433459/},
  doi = {10.1073/PNAS.70.4.1209}
}
</pre></td>
</tr>
<tr id="Singleton2010" class="entry">
	<td>Singleton AB, Hardy J, Traynor BJ and Houlden H (2010), <i>"Towards a complete resolution of the genetic architecture of disease"</i>, Trends in genetics : TIG.  Vol. 26(10), pp. 438. NIH Public Access.
	<p class="infolinks">[<a href="javascript:toggleInfo('Singleton2010','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Singleton2010','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1016/J.TIG.2010.07.004" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC2943029/ /pmc/articles/PMC2943029/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2943029/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Singleton2010" class="abstract noshow">
	<td><b>Abstract</b>: After years of linear gains in the genetic dissection of human disease we are now in a period of exponential discovery. This is particularly apparent for complex disease. Genome-wide association studies (GWAS) have provided myriad associations between common variability and disease, and have shown that common genetic variability is unlikely to explain the entire genetic predisposition to disease. Here we detail how one can expand on this success and systematically identify genetic risks that lead or predispose to disease using next-generation sequencing. Geneticists have had for many years a protocol to identify Mendelian disease. A similar set of tools is now available for the identification of rare moderate-risk loci and common low-risk variants. Whereas major challenges undoubtedly remain, particularly regarding data handling and the functional classification of variants, we suggest that these will be largely practical and not conceptual. textcopyright 2010 .</td>
</tr>
<tr id="bib_Singleton2010" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Singleton2010,
  author = {Singleton, Andrew B. and Hardy, John and Traynor, Bryan J. and Houlden, Henry},
  title = {Towards a complete resolution of the genetic architecture of disease},
  journal = {Trends in genetics : TIG},
  publisher = {NIH Public Access},
  year = {2010},
  volume = {26},
  number = {10},
  pages = {438},
  url = {/pmc/articles/PMC2943029/ /pmc/articles/PMC2943029/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2943029/},
  doi = {10.1016/J.TIG.2010.07.004}
}
</pre></td>
</tr>
<tr id="VanDenBogaert2003" class="entry">
	<td>Van Den Bogaert A, Schumacher J, Schulze TG, Otte AC, Ohlraun S, Kovalenko S, Becker T, Freudenberg J, J&ouml;nsson EG, Mattila-Evenden M, Sedvall GC, Czerski PM, Kapelski P, Hauser J, Maier W, Rietschel M, Propping P, N&ouml;then MM and Cichon S (2003), <i>"The DTNBP1 (Dysbindin) Gene Contributes to Schizophrenia, Depending on Family History of the Disease"</i>, The American Journal of Human Genetics., dec, 2003.  Vol. 73(6), pp. 1438-1443. Cell Press.
	<p class="infolinks">[<a href="javascript:toggleInfo('VanDenBogaert2003','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('VanDenBogaert2003','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1086/379928" target="_blank">DOI</a>]</p>
	</td>
</tr>
<tr id="abs_VanDenBogaert2003" class="abstract noshow">
	<td><b>Abstract</b>: We have investigated the gene for dystrobrevin-binding protein 1 (DTNBP1), or dysbindin, which has been strongly suggested as a positional candidate gene for schizophrenia, in three samples of subjects with schizophrenia and unaffected control subjects of German (418 cases, 285 controls), Polish (294 cases, 113 controls), and Swedish (142 cases, 272 controls) descent. We analyzed five single-nucleotide polymorphisms (P1635, P1325, P1320, P1757, and P1578) and identified significant evidence of association in the Swedish sample but not in those from Germany or Poland. The results in the Swedish sample became even more significant after a separate analysis of those cases with a positive family history of schizophrenia, in whom the five-marker haplotype A-C-A-T-T showed a P value of .00009 (3.1% in controls, 17.8% in cases; OR 6. 75; P = .00153 after Bonferroni correction). Our results suggest that genetic variation in the dysbindin gene is particularly involved in the development of schizophrenia in cases with a familial loading of the disease. This would also explain the difficulty of replicating this association in consecutively ascertained case-control samples, which usually comprise only a small proportion of subjects with a family history of disease.</td>
</tr>
<tr id="bib_VanDenBogaert2003" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{VanDenBogaert2003,
  author = {Van Den Bogaert, Ann and Schumacher, Johannes and Schulze, Thomas G. and Otte, Andreas C. and Ohlraun, Stephanie and Kovalenko, Svetlana and Becker, Tim and Freudenberg, Jan and J&ouml;nsson, Erik G. and Mattila-Evenden, Marja and Sedvall, G&ouml;ran C. and Czerski, Piotr M. and Kapelski, Pawel and Hauser, Joanna and Maier, Wolfgang and Rietschel, Marcella and Propping, Peter and N&ouml;then, Markus M. and Cichon, Sven},
  title = {The DTNBP1 (Dysbindin) Gene Contributes to Schizophrenia, Depending on Family History of the Disease},
  journal = {The American Journal of Human Genetics},
  publisher = {Cell Press},
  year = {2003},
  volume = {73},
  number = {6},
  pages = {1438--1443},
  doi = {10.1086/379928}
}
</pre></td>
</tr>
<tr id="Verlouw2021" class="entry">
	<td>Verlouw JA, Clemens E, de Vries JH, Zolk O, Verkerk AJ, am Zehnhoff-Dinnesen A, Medina-Gomez C, Lanvers-Kaminsky C, Rivadeneira F, Langer T, van Meurs JB, van den Heuvel-Eibrink MM, Uitterlinden AG and Broer L (2021), <i>"A comparison of genotyping arrays"</i>, European Journal of Human Genetics 2021 29:11., jun, 2021.  Vol. 29(11), pp. 1611-1624. Nature Publishing Group.
	<p class="infolinks">[<a href="javascript:toggleInfo('Verlouw2021','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Verlouw2021','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1038/s41431-021-00917-7" target="_blank">DOI</a>] [<a href="https://www.nature.com/articles/s41431-021-00917-7" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Verlouw2021" class="abstract noshow">
	<td><b>Abstract</b>: Array technology to genotype single-nucleotide variants (SNVs) is widely used in genome-wide association studies (GWAS), clinical diagnostics, and linkage studies. Arrays have undergone a tremendous growth in both number and content over recent years making a comprehensive comparison all the more important. We have compared 28 genotyping arrays on their overall content, genome-wide coverage, imputation quality, presence of known GWAS loci, mtDNA variants and clinically relevant genes (i.e., American College of Medical Genetics (ACMG) actionable genes, pharmacogenetic genes, human leukocyte antigen (HLA) genes and SNV density). Our comparison shows that genome-wide coverage is highly correlated with the number of SNVs on the array but does not correlate with imputation quality, which is the main determinant of GWAS usability. Average imputation quality for all tested arrays was similar for European and African populations, indicating that this is not a good criterion for choosing a genotyping array. Rather, the additional content on the array, such as pharmacogenetics or HLA variants, should be the deciding factor. As the research question of a study will in large part determine which class of genes are of interest, there is not just one perfect array for all different research questions. This study can thus help as a guideline to determine which array best suits a study's requirements.</td>
</tr>
<tr id="bib_Verlouw2021" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Verlouw2021,
  author = {Verlouw, Joost A.M. and Clemens, Eva and de Vries, Jard H. and Zolk, Oliver and Verkerk, Annemieke J.M.H. and am Zehnhoff-Dinnesen, Antoinette and Medina-Gomez, Carolina and Lanvers-Kaminsky, Claudia and Rivadeneira, Fernando and Langer, Thorsten and van Meurs, Joyce B.J. and van den Heuvel-Eibrink, Marry M. and Uitterlinden, Andr&eacute; G. and Broer, Linda},
  title = {A comparison of genotyping arrays},
  journal = {European Journal of Human Genetics 2021 29:11},
  publisher = {Nature Publishing Group},
  year = {2021},
  volume = {29},
  number = {11},
  pages = {1611--1624},
  url = {https://www.nature.com/articles/s41431-021-00917-7},
  doi = {10.1038/s41431-021-00917-7}
}
</pre></td>
</tr>
<tr id="Xie2015" class="entry">
	<td>Xie B, Agam G, Balasubramanian S, Xu J, Gilliam TC, Maltsev N and B&ouml;rnigen D (2015), <i>"Disease Gene Prioritization Using Network and Feature"</i>, Journal of Computational Biology., apr, 2015.  Vol. 22(4), pp. 313. Mary Ann Liebert, Inc..
	<p class="infolinks">[<a href="javascript:toggleInfo('Xie2015','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Xie2015','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1089/CMB.2015.0001" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC4808289/ /pmc/articles/PMC4808289/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4808289/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Xie2015" class="abstract noshow">
	<td><b>Abstract</b>: Identifying high-confidence candidate genes that are causative for disease phenotypes, from the large lists of variations produced by high-throughput genomics, can be both time-consuming and costly. The development of novel computational approaches, utilizing existing biological knowledge for the prioritization of such candidate genes, can improve the efficiency and accuracy of the biomedical data analysis. It can also reduce the cost of such studies by avoiding experimental validations of irrelevant candidates. In this study, we address this challenge by proposing a novel gene prioritization approach that ranks promising candidate genes that are likely to be involved in a disease or phenotype under study. This algorithm is based on the modified conditional random field (CRF) model that simultaneously makes use of both gene annotations and gene interactions, while preserving their original representation. We validated our approach on two independent disease benchmark studies by ranking candidate genes using network and feature information. Our results showed both high area under the curve (AUC) value (0.86), and more importantly high partial AUC (pAUC) value (0.1296), and revealed higher accuracy and precision at the top predictions as compared with other well-performed gene prioritization tools, such as Endeavour (AUC-0.82, pAUC-0.083) and PINTA (AUC-0.76, pAUC-0.066). We were able to detect more target genes (9/18/19/27) on top positions (1/5/10/20) compared to Endeavour (3/11/14/23) and PINTA (6/10/13/18). To demonstrate its usability, we applied our method to a case study for the prediction of molecular mechanisms contributing to intellectual disability and autism. Our approach was able to correctly recover genes related to both disorders and provide suggestions for possible additional candidates based on their rankings and functional annotations.</td>
</tr>
<tr id="bib_Xie2015" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Xie2015,
  author = {Xie, Bingqing and Agam, Gady and Balasubramanian, Sandhya and Xu, Jinbo and Gilliam, T. Conrad and Maltsev, Natalia and B&ouml;rnigen, Daniela},
  title = {Disease Gene Prioritization Using Network and Feature},
  journal = {Journal of Computational Biology},
  publisher = {Mary Ann Liebert, Inc.},
  year = {2015},
  volume = {22},
  number = {4},
  pages = {313},
  url = {/pmc/articles/PMC4808289/ /pmc/articles/PMC4808289/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4808289/},
  doi = {10.1089/CMB.2015.0001}
}
</pre></td>
</tr>
<tr id="Zheng2018" class="entry">
	<td>Zheng R, Li Z, He F, Liu H, Chen J, Chen J, Xie X, Zhou J, Chen H, Wu X, Wu J, Chen B, Liu Y, Cui H, Fan L, Sha W, Liu Y, Wang J, Huang X, Zhang L, Xu F, Wang J, Feng Y, Qin L, Yang H, Liu Z, Cui Z, Liu F, Chen X, Gao S, Sun S, Shi Y and Ge B (2018), <i>"Genome-wide association study identifies two risk loci for tuberculosis in Han Chinese"</i>, Nature Communications 2018 9:1., oct, 2018.  Vol. 9(1), pp. 1-9. Nature Publishing Group.
	<p class="infolinks">[<a href="javascript:toggleInfo('Zheng2018','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Zheng2018','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1038/s41467-018-06539-w" target="_blank">DOI</a>] [<a href="https://www.nature.com/articles/s41467-018-06539-w" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Zheng2018" class="abstract noshow">
	<td><b>Abstract</b>: Tuberculosis (TB) is an infectious disease caused by Mycobacterium tuberculosis (Mtb), and remains a leading public health problem. Previous studies have identified host genetic factors that contribute to Mtb infection outcomes. However, much of the heritability in TB remains unaccounted for and additional susceptibility loci most likely exist. We perform a multistage genome-wide association study on 2949 pulmonary TB patients and 5090 healthy controls (833 cases and 1220 controls were genome-wide genotyped) from Han Chinese population. We discover two risk loci: 14q24.3 (rs12437118, Pcombined = 1.72 × 10−11, OR = 1.277, ESRRB) and 20p13 (rs6114027, Pcombined = 2.37 × 10−11, OR = 1.339, TGM6). Moreover, we determine that the rs6114027 risk allele is related to decreased TGM6 transcripts in PBMCs from pulmonary TB patients and severer pulmonary TB disease. Furthermore, we find that tgm6-deficient mice are more susceptible to Mtb infection. Our results provide new insights into the genetic etiology of TB. Genetic risk loci for tuberculosis (TB) have so far been identified in African and Russian populations. Here, the authors perform a three-stage GWAS for TB in Han Chinese populations and find two risk loci near ESRRB and TGM6 and further demonstrate that tgm6 protects mice from Mtb infection.</td>
</tr>
<tr id="bib_Zheng2018" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Zheng2018,
  author = {Zheng, Ruijuan and Li, Zhiqiang and He, Fusheng and Liu, Haipeng and Chen, Jianhua and Chen, Jiayu and Xie, Xuefeng and Zhou, Juan and Chen, Hao and Wu, Xiangyang and Wu, Juehui and Chen, Boyu and Liu, Yahui and Cui, Haiyan and Fan, Lin and Sha, Wei and Liu, Yin and Wang, Jiqiang and Huang, Xiaochen and Zhang, Linfeng and Xu, Feifan and Wang, Jie and Feng, Yonghong and Qin, Lianhua and Yang, Hua and Liu, Zhonghua and Cui, Zhenglin and Liu, Feng and Chen, Xinchun and Gao, Shaorong and Sun, Silong and Shi, Yongyong and Ge, Baoxue},
  title = {Genome-wide association study identifies two risk loci for tuberculosis in Han Chinese},
  journal = {Nature Communications 2018 9:1},
  publisher = {Nature Publishing Group},
  year = {2018},
  volume = {9},
  number = {1},
  pages = {1--9},
  url = {https://www.nature.com/articles/s41467-018-06539-w},
  doi = {10.1038/s41467-018-06539-w}
}
</pre></td>
</tr>
<tr id="Zolotareva2019" class="entry">
	<td>Zolotareva O and Kleine M (2019), <i>"A Survey of Gene Prioritization Tools for Mendelian and Complex Human Diseases"</i>, Journal of integrative bioinformatics., sep, 2019.  Vol. 16(4) NLM (Medline).
	<p class="infolinks">[<a href="javascript:toggleInfo('Zolotareva2019','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Zolotareva2019','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1515/JIB-2018-0069/ASSET/GRAPHIC/J_JIB-2018-0069_FIG_002.JPG" target="_blank">DOI</a>] [<a href="https://www.degruyter.com/document/doi/10.1515/jib-2018-0069/html" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Zolotareva2019" class="abstract noshow">
	<td><b>Abstract</b>: Modern high-throughput experiments provide us with numerous potential associations between genes and diseases. Experimental validation of all the discovered associations, let alone all the possible interactions between them, is time-consuming and expensive. To facilitate the discovery of causative genes, various approaches for prioritization of genes according to their relevance for a given disease have been developed. In this article, we explain the gene prioritization problem and provide an overview of computational tools for gene prioritization. Among about a hundred of published gene prioritization tools, we select and briefly describe 14 most up-to-date and user-friendly. Also, we discuss the advantages and disadvantages of existing tools, challenges of their validation, and the directions for future research.</td>
</tr>
<tr id="bib_Zolotareva2019" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Zolotareva2019,
  author = {Zolotareva, Olga and Kleine, Maren},
  title = {A Survey of Gene Prioritization Tools for Mendelian and Complex Human Diseases},
  journal = {Journal of integrative bioinformatics},
  publisher = {NLM (Medline)},
  year = {2019},
  volume = {16},
  number = {4},
  url = {https://www.degruyter.com/document/doi/10.1515/jib-2018-0069/html},
  doi = {10.1515/JIB-2018-0069/ASSET/GRAPHIC/J_JIB-2018-0069_FIG_002.JPG}
}
</pre></td>
</tr>
<tr id="Zolotareva2019a" class="entry">
	<td>Zolotareva O and Kleine M (2019), <i>"A Survey of Gene Prioritization Tools for Mendelian and Complex Human Diseases"</i>, Journal of Integrative Bioinformatics., sep, 2019.  Vol. 16(4) De Gruyter.
	<p class="infolinks">[<a href="javascript:toggleInfo('Zolotareva2019a','abstract')">Abstract</a>] [<a href="javascript:toggleInfo('Zolotareva2019a','bibtex')">BibTeX</a>] [<a href="http://doi.org/10.1515/JIB-2018-0069" target="_blank">DOI</a>] [<a href="/pmc/articles/PMC7074139/ /pmc/articles/PMC7074139/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7074139/" target="_blank">URL</a>]</p>
	</td>
</tr>
<tr id="abs_Zolotareva2019a" class="abstract noshow">
	<td><b>Abstract</b>: Modern high-throughput experiments provide us with numerous potential associations between genes and diseases. Experimental validation of all the discovered associations, let alone all the possible interactions between them, is time-consuming and expensive. To facilitate the discovery of causative genes, various approaches for prioritization of genes according to their relevance for a given disease have been developed. In this article, we explain the gene prioritization problem and provide an overview of computational tools for gene prioritization. Among about a hundred of published gene prioritization tools, we select and briefly describe 14 most up-to-date and user-friendly. Also, we discuss the advantages and disadvantages of existing tools, challenges of their validation, and the directions for future research.</td>
</tr>
<tr id="bib_Zolotareva2019a" class="bibtex noshow">
<td><b>BibTeX</b>:
<pre>
@article{Zolotareva2019a,
  author = {Zolotareva, Olga and Kleine, Maren},
  title = {A Survey of Gene Prioritization Tools for Mendelian and Complex Human Diseases},
  journal = {Journal of Integrative Bioinformatics},
  publisher = {De Gruyter},
  year = {2019},
  volume = {16},
  number = {4},
  url = {/pmc/articles/PMC7074139/ /pmc/articles/PMC7074139/?report=abstract https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7074139/},
  doi = {10.1515/JIB-2018-0069}
}
</pre></td>
</tr>
</tbody>
</table>
<footer>
 <small>Created by <a href="http://jabref.sourceforge.net">JabRef</a> on 06/05/2022.</small>
</footer>
<!-- file generated by JabRef -->
</body>
</html>