# Analyzing the Environmental Legacy of Redlining in Google Earth Engine

Developed by **Zach Levitt**, '20.5 and **Dr. Jeff Howarth**, Associate Professor of Geography,
Middlebury College, Vermont, USA

This tutorial describes how to produce an analysis of redlining in an American city using modules developed in Google Earth Engine. For a description of the module functions themselves, go to the Modules tab.

## Introduction
This tutorial outlines a workflow for utilizing module functions in Google Earth Engine (GEE) to analyze and visualize the spatial legacy of redlining in U.S. cities. We use modules to define functions that can be applied to any city to produce layers that show land surface temperature, a vegetation index, and impervious surfaces within certain historical neighborhood boundaries. This tutorial offers descriptions of how to utilize the module functions defined here. To connect to the module functions defined here, add this line at the top of your script:

	// ------------------------------------------------
	// CONNECT TO MODULE FUNCTIONS
	// ------------------------------------------------
	var tools = require('users/zlevitt/chis:lst/lstModules.js');

Many of these methods are used in an example app that allows users to select a city and automatically calculate these data. This app can be found here.

## Outline
1. Define configuration dictionary
2. Choose a city of interest
3. Load environmental variables
4. Load HOLC data
5. Visualize data by neighborhood

## 1. Define configuration dictionary

	// ------------------------------------------------
	// This dictionary holds all of our variables
	// ------------------------------------------------
	var config = {
		currentCity: "",
		currentState: "",
		lstInstance: null,
		ndviInstance: null,
		imperviousInstance: null,
		cityPoint: null,
		RL_outline: null,
		NRL_outline: null,
		A_outline: null,
		B_outline: null,
		C_outline: null,
		holcFiltered: null,
		holc_A_B_C_table: null,
		holc_A_table: null,
		holc_B_table: null,
		holc_C_table: null,
		holc_D_table: null,
		holc_A_image: null,
		holc_B_image: null,
		holc_C_image: null,
		holc_D_image: null,
		holc_A_B_C_image: null,
		lst_NRL_masked: null,
		lst_RL_masked: null,
		ndvi_NRL_masked: null,
		ndvi_RL_masked: null,
		impervious_NRL_masked: null,
		impervious_RL_masked: null,
		A_TempPercDiff: 0,
		B_TempPercDiff: 0,
		C_TempPercDiff: 0,
		D_TempPercDiff: 0,
		A_NDVIPercDiff: 0,
		B_NDVIPercDiff: 0,
		C_NDVIPercDiff: 0,
		D_NDVIPercDiff: 0,
		A_ImperviousPercDiff: 0,
		B_ImperviousPercDiff: 0,
		C_ImperviousPercDiff: 0,
		D_ImperviousPercDiff: 0,
		maxImpervious: 0,
		minImpervious: 0,
		maxNDVI: 0,
		minNDVI: 0,
		minTemp: 0,
		maxTemp: 0,
		meanTemp: 0,
		meanImpervious: 0,
		meanNDVI: 0,
	};
	

## 2. Choose a city of interest
First, we need to create a dictionary of cities and geographic centroids:
	
	var cityDict = tools.createCityDict();
	// To view a list of cities, run print(cityDict);

Then, you can select a city you want to explore (replace 'Baltimore' with your choice):
	
	config.currentCity = 'Baltimore';

Once you select a city, use this line to select the centroid of your city:

	config.cityPoint = ee.Geometry(cityDict.get(config.currentCity));
	

## 3. Load environmental variables
These lines call upon modules developed by Ermida et al (2020):

	// ------------------------------------------------
	// Calculate environmental variables
	// ------------------------------------------------
	config.lstInstance = tools.loadLST_2016to2020_max(config.cityPoint);
	config.ndviInstance = tools.loadNDVI_2016to2020_max(config.cityPoint);
	config.imperviousInstance = tools.loadImpervious_NLCD2016_max(config.cityPoint);

## 4. Load HOLC data
This line generates HOLC data for your city:

	// ------------------------------------------------
	// Load HOLC data
	// ------------------------------------------------
	config = tools.loadHOLC(config);

## 5. Visualize data by neighborhood
#### Charts
	config = tools.calculateStats(config);
	var arrayTemp = ee.Array([[config.A_TempPercDiff],[config.B_TempPercDiff],[config.C_TempPercDiff],[config.D_TempPercDiff]]);
	tempStatsChart = tools.createPercDiffChart(arrayTemp,'Land Surface Temperature (F)',config.minTemp,config.maxTemp);
	print(tempStatsChart);

	var arrayNDVI = ee.Array([[config.A_NDVIPercDiff],[config.B_NDVIPercDiff],[config.C_NDVIPercDiff],[config.D_NDVIPercDiff]]);
	NDVIStatsChart = tools.createPercDiffChart(arrayNDVI,'NDVI (Tree canopy and green space)',config.minNDVI,config.maxNDVI);
	print(NDVIStatsChart);

	var arrayImpervious = ee.Array([[config.A_ImperviousPercDiff],[config.B_ImperviousPercDiff],[config.C_ImperviousPercDiff],[config.D_ImperviousPercDiff]]);
	ImperviousStatsChart = tools.createPercDiffChart(arrayImpervious,'Impervious surfaces (roads and buildings)',config.minImpervious,config.maxImpervious);
	print(ImperviousStatsChart);

#### Map

	var lstPalette = ['#005b7b','#3897bc','#77b1c7','#b4c6cd','#ddb588','#f19a3f','#d6721f','#b13b03','#750401'];
	var lstVis = {
		min: config.meanTemp.int().subtract(15).getInfo(),
		max: config.meanTemp.int().add(15).getInfo(),
		palette: lstPalette
	};
	map.centerObject(config.cityPoint,12);
	map.addLayer(config.lstInstance,lstVis, 'LST',true,0.4);
	map.addLayer(config.ndviInstance,tools.treeVis, 'NDVI',false,1);
	map.addLayer(config.imperviousInstance,tools.imperviousVis, 'Impervious',false,1);
	map.addLayer(config.lst_RL_masked,lstVis, 'Redlined neighborhoods image',true,1);//4
	map.addLayer(config.lst_NRL_masked,lstVis, 'Not redlined neighborhoods image',true,1);//5
	map.addLayer(config.NRL_outline, {palette: '#000000'}, 'Not redlined neighborhoods - outline', true, 1);//6
	map.addLayer(config.holc_A_image, tools.holcVis, 'Grade A image', false, 0.8);//7
	map.addLayer(config.holc_B_image, tools.holcVis, 'Grade B image', false, 0.8);//8
	map.addLayer(config.holc_C_image, tools.holcVis, 'Grade C image', false, 0.8);//9
	map.addLayer(config.holc_D_image, tools.holcVis, 'Grade D image', false, 0.8);//10
	map.addLayer(config.A_outline, {palette: '#3e911d'}, 'Grade A outline', false, 1);//11
	map.addLayer(config.B_outline, {palette: '#4689bf'}, 'Grade B outline', false, 1);//12
	map.addLayer(config.C_outline, {palette: '#ebd10e'}, 'Grade C outline',false, 1);//13
	map.addLayer(config.RL_outline, {palette: '#eb0000'}, 'Redlined neighborhoods - outline', 1, 1);
  
