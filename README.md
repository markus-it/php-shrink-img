# php-shrink-img  PHP Script who shrink images
<?php
include('function_thumb.php');
/*
*
* bild uploaden
*
*/
if(isset($_POST['upload']))
{
 //error array initialisieren
 $error = array();
 
 //kompletter string der datei aus dem datei upload feld
 $image_string = $_FILES['img']['name'];
 
 $img_kb = $_FILES['img']['size']; //bildgröße des hochzuladenden bildes holen
 $image_size = 2000 * 1024; //maximale datei größe in kb festlegen
 
 $max_breite = 1024; //maximal zulässige breite des bildes festlegen
 $max_hoehe = 768; //maximal zulässige höhe des bildes festlegen
 
 $image_mase = getimagesize($_FILES['img']['tmp_name']); //array bilden mit breite und höhe des bildes
 
 $img_breite = $image_mase[0]; //breite des hochzuladenden bildes holen
 $img_hoehe = $image_mase[1]; //höhe des hochzuladenden bildes holen
 
 /*
 ------------------------------------------------------------------------
  ermitteln des Dateinamens nach dem letzten im string vorkommenden
  punkt
 ------------------------------------------------------------------------
 */
 $image_end = strrchr($image_string,'.');
 
 /*
 ------------------------------------------------------------------------
  ermitteln der zeichenlänge des strings der aus strrchr übrig
  bleibt und die dateiendung wieder spiegelt
 ------------------------------------------------------------------------
 */
 $sub_count = strlen($image_end);
 
 /*
 ------------------------------------------------------------------------
  hier holen wir uns nun den datei namen, den brauchen wir für die 
  bevorstehende namens prüfung auf zeichenvorkommnisse
 ------------------------------------------------------------------------
 */
 $image_name = substr($image_string, 0, -$sub_count);
 
 //prüfen magic_quotes on
 if(get_magic_quotes_gpc()==1 && get_magic_quotes_runtime())
 {
  $image_name = stripslashes($image_name);
  $image_end = stripslashes($image_end);
 }
 
 //prüfen ob checkbox angehakt ist
 if(isset($_POST['edit_ok']) !=1)
 {
  $error['pwe_false'] = 'Vor dem ändern bitte Haken un Box setzen!';
 }
 
 /*
 ------------------------------------------------------------------------
  whitelist endungs array für bilder
 ------------------------------------------------------------------------
 */
 $end_arr = array('.jpg','.JPG','.jpeg','.png');
 
 /*
 ------------------------------------------------------------------------
  bilddatei gegen die whitelist der bilderendungen checken
 ------------------------------------------------------------------------
 */
 if(!in_array($image_end,$end_arr))
 {
  $error['end_false'] ='Die Endung des Bildes hat ein unerwünschtes Format!';
 }
 
 /*
 ------------------------------------------------------------------------
  prüfen des datenamens auf verbotene zeichen es dürfen nur zeichen
  a-z A-Z 0-9 . _ - vorkommen
 ------------------------------------------------------------------------
 */
 if(!preg_match("#^([a-z0-9\._-]+)$#si",$image_name))
 {
  $error['name_false'] ='Der Bildname enthält verbotene Zeichen!';
 }
 
 /*
 ------------------------------------------------------------------------
  prüfen ob die breite des bildes größer als der maximal zulässige
  breitenwert ist
 ------------------------------------------------------------------------
 */
 if($img_breite > $max_breite)
 {
  $error['breite_false'] ='das bild ist zu breit maximal '.$max_breite.' px';
 }
 
 /*
 ------------------------------------------------------------------------
  prüfen ob die hoehe des bildes größer als der maximal zulässige
  hoehenwert ist
 ------------------------------------------------------------------------
 */
 if($img_hoehe > $max_hoehe)
 {
  $error['hoehe_false'] ='das bild ist zu hoch maximal '.$max_hoehe.' px';
 }
 
 /*
 ------------------------------------------------------------------------
  prüfen der maximal zulässigen kb größe des bildes
  2 MB maximal
 ------------------------------------------------------------------------
 */
 if($img_kb > $image_size)
 {
  $error['size_false'] ='das bild ist zu groß maximal '.$image_size.' kb';
 }
 
 //keine fehler dann uploaden und in db schreiben
 if(!$error)
 {  
  //prüfen ob das bild hochgeladen wurde und dann kleines bild kopieren
  if(!is_uploaded_file($_FILES['img']['name']))
  {
   //upload ausführen
   move_uploaded_file($_FILES['img']['tmp_name'],"bilder/".$_FILES['img']['name']);
   
   $bild = $_FILES['img']['name'];
   $src ="bilder/".$_FILES['img']['name']."";
   $thumb_order ="user/profil_bild/";
   $maximal ="140";
   $thumb_name = $_FILES['img']['name'];
   
   thumbnail($src,$maximal,$thumb_order,$thumb_name);
  }  
 }
} 



Anpassung Bilder Grösse
 <?php
/*
*
* funktion für das erstellen von thumbnails
*
*/
if(!function_exists('thumbnail')){
 
 function thumbnail($img,$maximal,$thumb_order,$thumb_name){ //parameter für anwendung der function
 
  $size = getimagesize($img); //ermitteln der bilddaten, höhe,breite und dateityp
  
  //liste der ermittelten bilddaten erstellen, achtung werte
  //werden von rechts nachlinks gelsen
  list($breite,$hoehe,$datentyp) = $size;
           
  //ermitteln ob höhe oder breite des bildes größer sind und herunterrechnen
  if($breite > $maximal || $hoehe > $maximal){
  
   //brechnungsfaktor festlegen
   $rechen_faktor = $breite / $hoehe;
   
   //ist der faktor kleiner 1 dann neue breite errechnen
   if($rechen_faktor < 1){
   
    $neue_breite = $maximal * $rechen_faktor;
    $neue_hoehe = $maximal;
   }
   
   //ist der faktor größer oder gleich 1 dann neue höhe errechnen
   if($rechen_faktor >= 1){
   
    $neue_hoehe = $maximal / $rechen_faktor;
    $neue_breite = $maximal;
   
   }
   
   //thumbnail mit neu errechneten werten anzeigen
   $thumbnail = imagecreatetruecolor($neue_breite,$neue_hoehe);
   
   //bilder nach bilddatentyp ermitteln mit switch konstrukt nur auf jpg und png
   //ausgerichtet
   switch($datentyp){
    
    case 2:
     $image = imagecreatefromjpeg($img);
    break;
    
    case 3:
     $image = imagecreatefrompng($img);
    break;
    
   }//ende switch
   
   //neues bild mit allen ermittelten daten in neues verzeichnis kopieren
   imagecopyresized($thumbnail,$image, 0, 0, 0, 0, $neue_breite, $neue_hoehe, $breite, $hoehe);
   
   //neues erstelltes thumbnail nach ermittelten datentyp erstellen
   switch($datentyp){
    
    case 2:
     imagejpeg($thumbnail,$thumb_order.'thumbnail_'.$thumb_name);
    break;
    
    case 3:
     imagepng($thumbnail,$thumb_order.'thumbnail_'.$thumb_name);
    break;
    
   }//ende switch
   
   //rückgabewert der funfunktion
   return 1;
   
  }// ende der seitenrechnung
 }//ende der funktion
}// ende prüfen ob function schon existiert
?> 
