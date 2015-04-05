<?php
include('../parseMaster.php');
include('api/Meetup.php');
include('api/MeetupApiRequest.class.php');
include('api/MeetupApiResponse.class.php');
include('api/MeetupEvents.class.php');
include('api/MeetupConnection.class.php');
define( 'PARSE_SDK_DIR', '../../Parse/src/Parse/' );
require_once( '../../Parse/autoload.php' );
use Parse\ParseClient;
use Parse\ParseObject;
use Parse\ParseQuery;
use Parse\ParseUser;
use Parse\ParseGeoPoint;
use Parse\ParseFile;
use Parse\ParseException;
ParseClient::initialize( $app_id, $rest_key, $master_key );
include('../eventbrite/EventbriteMaster.php'); 
$eb_client = new Eventbrite( array('app_key'=>'ZO2DNNLNEKI4RDA2LF', 'user_key'=>'137952599773362632609'));                                                                              
$eb_client = new Eventbrite( array('access_token'=>'ZPU4IH7ZU3ANCXYNRANQ'));

$cats = array(
	//array(999,"Popular Events (Can't select categories)"),
	array(101,"Nightlife"),
	array(102,"Happy Hour"),
	array(103,"Entertainment"),
	array(104,"Music"),
	array(105,"Dining"),
	array(106,"Fundraiser"),
	array(107,"Sports"),
	array(108,"Shopping"),
	array(109,"Meetup"),
	array(110,"Freebies"),
	array(111,"Other")
);
$keywordLibrary = array(
    //999 => ["popular"],
	101 => [1],
	102 => [1],
	103 => [5],
	104 => [21],
	105 => [10],
	106 => [13],
	107 => [32],
	108 => [8],
	109 => [31],
	110 => [15],
	111 => ["other"]

);
$displayPage = array(
	"date" => "DATE / TIME",
	"ebID" => "EB",
	"url" => "URL",
	"description" => "DESCRIPTION"
);
function objectToArray($d) {
	if (is_object($d)) {
		$d = get_object_vars($d);
	}
	if (is_array($d)) {
		return array_map(__FUNCTION__, $d);
	}
	else {
		return $d;
	}
}

//Meetup find all open events given the parameters
//Parameters required to return events on Meetup API console: city, country, state, category

function getOpenEvents( $Parameters ) {
    $required_params = array( 'city', 'country', 'lat', 'lon', 'state', 'text', 'topic', 'zip' );
    $url = $this->buildUrl( MEETUP_ENDPOINT_OPEN_EVENTS, $Parameters, $required_params );
    $response =  $this->get( $url )->getResponse();
    return $response['results'];
}
    
//Example usage:

$api_key = '1fa783d777a126641742a3042294a75';
$connection = new MeetupKeyAuthConnection($api_key);    
    
$m = new MeetupEvents($connection);
$events = $m->getOpenEvents( array( 'zip' => '20052'));

foreach($events as $e) {
    echo $e['name'] . " at " . date(DATE_W3C, $e['time']/1000) . "<br>";
}

$events = array();
function searchEvent($events,$city,$max,$categories,$startEnd){
	global $keywordLibrary;
	global $cats;
	global $eb_client;
	$keywords = "";
	foreach ($categories as $category){
		$temp = "";
		foreach ($keywordLibrary[$category] as $catTemp){
			$catTemp = str_replace(' ', '%20', $catTemp);
		    if ($catTemp == "popular") {
		        $showPopular = true;
		    } else if (sizeof($categories) == 1){
				$temp.=$catTemp;
			} else {
				$temp.=$catTemp."%20OR%20";
			}
		}
		$keywords.=$temp;
	}
	if (sizeof($categories) != 1){
		$keywords = substr($keywords, 0, -8);
	}
	
	    $search_params = array(
	        'max' => $max,
	        'city' => $city,
	        'country' => 'US',
		    'keywords' => $keywords,
		    'date' => $startEnd,
		   /* 'sort_by' => 'date' */
	    );

	//$resp = $eb_client->event_search( $search_params );
	//$m = new MeetupEvents($connection);
    $resp = $m->getOpenEvents( array( 'zip' => '20052', ));
    $resp = objectToArray($resp);
    
	$show = ($resp["events"][0]["summary"]["num_showing"]);
	$i = 0;
	while($i < $show){
		foreach ($resp as $event){
            $i++;
            $newEvent = array(
            	"ebID" => $event[$i]["event"]["id"], 
            	"category" => $categories[0],
            	"title" => $event[$i]["event"]["title"],
            	"date" => $event[$i]["event"]["start_date"],
            	"endDate" => $event[$i]["event"]["end_date"],
            	"location" => $event[$i]["event"]["venue"]["name"],
            	"lon" => $event[$i]["event"]["venue"]["longitude"],
            	"lat" => $event[$i]["event"]["venue"]["latitude"],
            	"organizer" => $event[$i]["event"]["organizer"]["name"],
            	"organizerID" => $event[$i]["event"]["organizer"]["id"],
            	"image" => $event[$i]["event"]["logo"],
            	"url" => $event[$i]["event"]["url"],
            	"description" => strip_tags($event[$i]["event"]["description"])
            );
            /*$checkEB = new ParseQuery("Event");
            $checkEB->equalTo("ebID", (string)$newEvent["ebID"]);
			$checkEBRes = $checkEB->find();*/
			$testerEB = array();
			foreach ($events as $evnt){
				$testerEB[] = $evnt["ebID"];
			}
			$events[] = $newEvent;
		}
	}
	return $events;
}
if (isset($_POST['form-submit'])){
	$city = $_POST['city'];
	$max = $_POST['max-events'];
	$categories = $_POST['categories'];
	$start = $_POST['start-date'];
	$end = $_POST['end-date'];
	$error = false;
	if (empty($categories)){
		$errorCats = true;
		$error = true;
	}
	if (empty($max)){
		$errorMax = true;
		$error = true;
	}
	if (!$error){
		$count = 0;
		$startEnd = $start." ".$end;
		while ($count != $max) {
			$events = searchEvent($events,$city,$max,$categories,$startEnd);
			$count = (count($events));
			//$count++;
		}
		$found = true;
	}
}
if (isset($_POST['form-submit-2'])){
	$size = sizeof($_POST['inputCheckbox']);
	$checks = $_POST['inputCheckbox'];
	for($ref = 0; $ref < $size; $ref++){
		$eventParse = new ParseObject("Event");
	    $eventParse->set("ebID", $_POST['ebID'][$checks[$ref]]);
	    $eventParse->set("Title", $_POST['title'][$checks[$ref]]);
	    $eventParse->set("Date", new DateTime($_POST["date"][$checks[$ref]]));
	    $eventParse->set("EndTime", new DateTime($_POST["endDate"][$checks[$ref]]));
	    $eventParse->set("CreatedByName", $_POST["organizer"][$checks[$ref]]);
	    $eventParse->set("TicketLink", $_POST["url"][$checks[$ref]]);
	    $eventParse->set("URL", $_POST["url"][$checks[$ref]]);
	    $eventParse->set("Location", $_POST["location"][$checks[$ref]]);
	    $eventParse->set("Description", str_replace(array("\r\n", "\r"), "\n", strip_tags($_POST["description"][$checks[$ref]])));
	    
	    $lat = floatval($_POST["lat"][$checks[$ref]]);
	    $long = floatval($_POST["lon"][$checks[$ref]]);
	    $point = new ParseGeoPoint($lat, $long);
		$eventParse->set("GeoLoc", $point);
		
		$hashtag = "";
		foreach ($cats as $cat){
			if ((string)$cat[0] == (string)$_POST["eventCategory"][$checks[$ref]]){
				$hashtag = $cat[1];
				break;
			}
		}
		
	    $eventParse->set("Hashtag", $hashtag);
	    
	    $imageChecks = $_POST['imageCheck'];
	    if (in_array($ref, $imageChecks)) {
		    $localFilePath = $_POST["image"][$checks[$ref]];
		    if ($localFilePath) {
		        $file = ParseFile::createFromFile($localFilePath, "image.jpg");
		        $eventParse->set("Image", $file);
		    }
	    }
	    
	    $orgID = 'eb'.$_POST["ebID"][$checks[$ref]].'';
	    $eventParse->set("CreatedBy", $orgID);
	    
	    $eventParse->set("swipesRight", 1);
	    $eventParse->set("swipesLeft", 0);
	    $eventParse->set("eventBrite", true);
	    
	    $checkEB = new ParseQuery("Event3");
        $checkEB->equalTo("ebID", (string)$_POST['ebID'][$checks[$ref]]);
		$checkEBRes = $checkEB->find();
	    if ($checkEB->count() == 0){
		    try {
		      $eventParse->save();
		    } catch (ParseException $ex) {  }
	    }
	}
	header("Location: http://happening.city/admin");
}
?>
