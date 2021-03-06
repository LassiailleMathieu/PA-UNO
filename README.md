# ARDUINOPEN
# Projet Arduino 2019-2020

Author: Lassiaille Mathieu

Email: mlassiaille12@gmail.com

Date: du 01/12/19

Revision: version1

License: Public Domain

Project: Arduinopen

#### 1.Description du projet: 


    Mon projet consiste à créer un systeme de verouillage pour n'importe quelle porte, 
    qui ne s'ouvre que pour une séquence de frappes scécifique, définie préalablement par le propriétaire.

#### 2. Step 1: Mon github
Sur mon github vous trouverez:

1. Mes rapports de scéances
2. Mon CDC
3. Un fichier décrivant les fonctions principales de mon code
4. Un fichier contenant le code final et complet (pas encore disponible)
5. Ce fichier !
6. Un fichier contenant de la documentation supplémentaire

#### 3. Step 2: Photo du projet final

https://github.com/LassiailleMathieu/PA-UNO/blob/master/Documentation/Images/imageProjetArduino.jpg

#### 4. Step 3: Le code

    /*
    Ce programme est ma version finale
   */

    //Pin

    const int Piezo = A5 ; //exemple de port sur lequel sera branché le piezo
    const int BoutonPoussoir = 6 ; //port sur lequel est branché le bouton poussoir
    const int servoPin = 9;   // servo connecté au pin 9

    //variables de réglages

    const int seuil=90; //seuil de déclenchement, on veut pas que notre programme enregistre de simple vibrations comme des frappes, il faut donc frapper bien fort pour qu'il s'active
    const int nombreMaxDeFrappes = 5 ; //On donne a notre régleur un nombre max de 15 frappes pour son code secret
    const int tempsMaxDeFrappe = 3500 ; //Son code doit pas durer trop longtemps
    const int differenceLocaleMax = 30 ;


    //variables globales

    int tabIntervalEntreFrappes[nombreMaxDeFrappes] ; //on note les intervalles de temps dans ce tableau, on les normalise pas encore
    int tabIntervalEntreFrappesCS[nombreMaxDeFrappes] = { 214 , 258 , 242 , 0 , 0 } ;
    long tabFrappe[nombreMaxDeFrappes] ; //on note les temps normalisé d'une frappe quelconque 
    long tabCodeSecret[nombreMaxDeFrappes] ; //on note les temps normalisé à 

    int valeurPiezo ; //on stoque  la valeur du Piezo dans cette variable
    int valeurBoutonPoussoir ; //on stoque  la valeur du bouton poussoir dans cette variable

    void setup() {
        Serial.begin(9600) ;
        pinMode(BoutonPoussoir, INPUT) ; //le bouton poussoir est in INPUT
    }

    void loop() {
      valeurPiezo = analogRead(Piezo) ;
      valeurBoutonPoussoir = digitalRead(BoutonPoussoir) ;
  
      if (seuil <= valeurPiezo) {
        if ( valeurBoutonPoussoir == HIGH) {
        ecouterFrappeCS() ;
      }
    else {
        ecouterFrappe() ;
        normaliserFrappe() ;
        if (validerFrappe()) {
         servowrite(servoPin, 0, 500, 2400); 
         delay(500) ;
         servowrite(servoPin, 90, 500, 2400); 
         servowrite(servoPin, 180, 500, 2400); 
        }
        else {
          Serial.println("^^^^^^^^^^^^^^") ;
          Serial.println( "la porte ne souvre pas") ;
          Serial.println("^^^^^^^^^^^^^^") ;
        }
      }
     }
    }

    void ecouterFrappe() {
  
       Serial.println("Ca commence") ;
  
       //Tout d'abord on remet le tableau a 0
       for (int i=0;i<nombreMaxDeFrappes ; i++){
          tabIntervalEntreFrappes[i]=0 ;
       }
  
       int nombreDeFrappes = 0 ;
       int tempsDeDebut = millis() ;
       int tempsPresent = millis() ;


    delay(150) ;
 
  
    do{
    
    valeurPiezo = analogRead(Piezo) ;//la valeur est stockée
    
    if (seuil <= valeurPiezo) { //on commence à enregistrer que si le seuil est atteint
      Serial.println("frappe"); //on indique qu'on a une frappe

      tempsPresent=millis() ; //on enregistre le temps entre le début et la premiere frappe
      Serial.println(tempsPresent-tempsDeDebut);
      tabIntervalEntreFrappes[ nombreDeFrappes ] = tempsPresent-tempsDeDebut ; // on note cet écart dans la liste 
      nombreDeFrappes++ ; // on incrémente notre nombe de frappe
      
      tempsDeDebut=tempsPresent ; //on remet la différence a 0ms 
      delay(150) ;
    }
    tempsPresent= millis() ;
    
    }while ( (nombreDeFrappes <= (nombreMaxDeFrappes-1)) && ((tempsPresent-tempsDeDebut)<=tempsMaxDeFrappe));
    Serial.println("C'est finiiiiiiiiiiiiiiiiiiii") ;
    Serial.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~") ;
    for (int i=0 ; i < nombreMaxDeFrappes ; i++) {
      Serial.println(tabIntervalEntreFrappes[i]) ;
    }
    Serial.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~") ;
    Serial.println("fin de la fonction ecouterFrappe()") ;
    Serial.println("début de la fonction normaliserFrappe()") ;
    }

    void ecouterFrappeCS() {

     Serial.println("Ca commence code secret") ;

     //Tout d'abord on remet le tableau a 0
     for (int i = 0; i < nombreMaxDeFrappes ; i++) {
       tabIntervalEntreFrappesCS[i] = 0 ;
     }

    int nombreDeFrappes = 0 ;
    int tempsDeDebut = millis() ;
    int tempsPresent = millis() ;


    delay(150) ;

 
    do {

    valeurPiezo = analogRead(Piezo) ;//la valeur est stockée

    if (seuil <= valeurPiezo) { //on commence à enregistrer que si le seuil est atteint
      Serial.println("frappe"); //on indique qu'on a une frappe

      tempsPresent = millis() ; //on enregistre le temps entre le début et la premiere frappe
      Serial.println(tempsPresent - tempsDeDebut);
      tabIntervalEntreFrappesCS[ nombreDeFrappes ] = tempsPresent - tempsDeDebut ; // on note cet écart dans la liste
      nombreDeFrappes++ ; // on incrémente notre nombe de frappe

      tempsDeDebut = tempsPresent ; //on remet la différence a 0ms
      delay(150) ;
    }
    tempsPresent = millis() ;

    } while ( (nombreDeFrappes <= (nombreMaxDeFrappes - 1)) && ((tempsPresent - tempsDeDebut) <= tempsMaxDeFrappe));
    Serial.println("C'est finiiiiiiiiiiiiiiiiiiiiiiiiiiiiiii");
    Serial.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");
    for (int i = 0 ; i < nombreMaxDeFrappes ; i++) {
    Serial.println(tabIntervalEntreFrappesCS[i]) ;
    }
    Serial.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");
    Serial.println("code secret enregistré ;) ") ;
    }

    void normaliserFrappe() {

      int intervalMaxEntreFrappesCS = 0 ;
      int intervalMaxEntreFrappes = 0 ;

     //on parcour notre tableau de frappe pour trouver l'écart max qui deviendra notre réference
     for (int i = 0 ; i < (nombreMaxDeFrappes) ; i++) {
       if ( tabIntervalEntreFrappes[i] > intervalMaxEntreFrappes) {
          intervalMaxEntreFrappes = tabIntervalEntreFrappes[i] ;
       }
       if ( tabIntervalEntreFrappesCS[i] > intervalMaxEntreFrappesCS) {
          intervalMaxEntreFrappesCS = tabIntervalEntreFrappesCS[i] ;
       }
    }
     Serial.println("***************") ;
    for (int i = 0; i < (nombreMaxDeFrappes) ; i++) {
    tabCodeSecret[i] = map(tabIntervalEntreFrappesCS[i],0,intervalMaxEntreFrappesCS,0,100) ;
    tabFrappe[i] = map(tabIntervalEntreFrappes[i],0,intervalMaxEntreFrappes,0,100) ;
    Serial.println(String(tabFrappe[i])) ;
    }
     Serial.println("***************") ;  
  
    /* Les tableaux 'tabFrappe[]' et 'tabCodeSecret[]' contiennent  
    *  maintenant les écarts normalisées.
    *  Ils sont prêt à etre comparés!
    */
    }

    boolean validerFrappe() {

      int nombreFrappesEssai = 0 ;
      int nombreFrappesCS = 0 ;

     int differenceDeTempsLocale = 0 ;
  
     //on commence d'abord par contabiliser le nombre de frappe 
    for (int i=0; i<nombreMaxDeFrappes ; i++) {
    if (tabFrappe[i] > 0) {
      nombreFrappesEssai++ ;
    }
    if (tabCodeSecret[i] > 0) {
      nombreFrappesCS++ ;
    }
    }

    //on fait le test le plus simple: est ce qu'ils ont le même nombre kkde frappes
    if (nombreFrappesEssai!=nombreFrappesCS) {
      Serial.println("il n'y a pas le même nombre de frappes") ;
      return false ;
    }

    //on verifie chaque frappe individuelle et on calcul l'ecart totale
    for (int i=0; i<nombreMaxDeFrappes ; i++) {
      differenceDeTempsLocale = abs(tabCodeSecret[i]-tabFrappe[i]) ;
      if (differenceDeTempsLocale > differenceLocaleMax ) {
        Serial.println("Une frappe n'est pas normale") ;
        return false ;
    }
    }
  
    Serial.println("La frappe est validée <3") ;
    return true ;
    }

    void servowrite(int pin, int angle, int microzero, int micro180){
      // pin for servo, angle to move, microseconds for zero degrees, microseconds for 180 degrees
      int microvalue= map(angle, 0, 180, microzero, micro180); //map angle to microseconds
      for (int i = 0; i < 50; i++)   //Send 50 pwm pulses with width microvalue uS
      {
        analogWrite(servoPin,255);              //Pulse high for microvalue uS     
        delayMicroseconds(microvalue);
        analogWrite(servoPin,0);                //Pulse low for 15mS
        delay(15);
       }
     delay(10);            //Pause
   }



#### 5. Contributions
Référent: Pascal Masson ; http://users.polytech.unice.fr/~pmasson/Enseignement-arduino.htm

Profile Arduino: https://id.arduino.cc/LassiailleMathieu

Merci à tous ceux qui ont contribué au projet, notament Thomas Prevost ;)





