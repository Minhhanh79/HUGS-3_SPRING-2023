#include <ArduinoBLE.h>

#define SERVICE_UUID "19B10000-E8F2-537E-4F6C-D104768A1214" //uart service UUID
//#define CHARACTERISTIC_UUID_RX "6E400002-B5A3-F393-E0A9-E50E24DCCA9E" //receive
#define CHARACTERISTIC_UUID_TX "19B10001-E8F2-537E-4F6C-D104768A1214" //transfer

const unsigned long READ_PERIOD = 100000; //4000us; 250Hz -> 480, 33200 us; 30Hz

BLEService Dataservice(SERVICE_UUID);
BLECharacteristic Datachar (CHARACTERISTIC_UUID_TX, BLERead   | BLEWrite, 3);

//#define number of sensor
int n = 1;

int portvalue[1000];
byte hilo[2000];

//int newInt;

//byte hi, lo;

void setup(){
  
  Serial.begin(115200);
  
  // while(!Serial);

  if(!BLE.begin()){
    Serial.println("starting BLE failed.");
    while(1);
  }
  
  BLE.setLocalName("Arduino Nano 33 IoT");
  BLE.setAdvertisedService(Dataservice); // add the service UUID
  
  Dataservice.addCharacteristic(Datachar); // add the battery level characteristic
  
  BLE.addService(Dataservice); // Add the battery service
  
  for (int i = 0; i < n; i++){
    portvalue[i] = 0;
    }
    
  Datachar.writeValue(portvalue, n); // set initial value for this characteristic
  
  BLE.advertise();
  Serial.println("Bluetooth device active, waiting for connections...");
}

void loop(){
  BLEDevice central = BLE.central(); // wait for a BLE central

  if(central){
    Serial.println("Connected to central device!");
    Serial.println(central.address()); // print the central's BLE address:
    Serial.println(" ");

    // if a central is connected to the peripheral:
    while(central.connected()){
      static unsigned long lastRead;
      if (micros() - lastRead >= READ_PERIOD)
      {
        lastRead += READ_PERIOD;
        int j = 0;
        for (int i = 0; i < n; i++){
          portvalue[i] = byte(analogRead(i));
          hilo[j] = highByte(portvalue[i]);
          j=j+1;
          hilo[j] = lowByte(portvalue[i]);
          j=j+1;
        }
     
        
        Datachar.writeValue(hilo, 2*n);
        for (int i = 0; i < 2*n; i=i+2) {
          Serial.print(hilo[i]);
          Serial.print(" ");
          Serial.print(hilo[i+1]);
          Serial.print(" ");
          Serial.println((hilo[i] << 8) + hilo[i+1]);
        }
        Serial.println();
      }
    }
    Serial.print("Disconnected from central: ");
    Serial.println(central.address());
  }
}



