// Output a Teddy Ruxpin automation stream
//
// 2024, feraloid

// Teddy Ruxpin automation data is carried on the RIGHT cassette channel, audio is on the left channel
// The automation data is in packets or frames repeated at a rate of approximately 18ms
// Each data packet contains data for all 8 servo channels
// The servo data is encoded into the gap length between clock pulses
// Channel 5 switches audio output between Teddy and Grubby, if Grubby is connected
// From observing the data stream, it appears that a minimum servo value (630us) routes audio to Teddy,
//  while a maximum servo value (1590us) routes audio to Grubby
// (I don't have a Grubby, so I can't confirm details of Grubby's operation)
//
// Some information was gleaned from https://github.com/furrtek/Svengali
// Clock pulses = 400us, positive
// Servo pulses = 630-1590us, negative (gaps between clock pulses)
// Servo range  = 960us
// 
// Channel 1: unused
// Channel 2: Teddy's eyes
// Channel 3: Teddy's upper jaw
// Channel 4: Teddy's lower jaw
// Channel 5: Audio Routing  -- determines if audio plays on Teddy or Grubby?
// Channel 6: Grubby's eyes
// Channel 7: Grubby's upper jaw
// Channel 8: Grubby's lower jaw
//
// channel 2 min value = eyes wide open; max value - eyes fully closed
// channel 3 & 4 min value = upper and lower jaws fully closed; max value = upper and lower jaws wide open
//
// Signal construction:
// C=clock
// #=Servo channel signal (gap length=servo value)
//   _      _      _      _      _      _      _      _      _
//__| |____| |____| |____| |____| |____| |____| |____| |____| |____
//   C  1   C  2   C  3   C  4   C  5   C  6   C   7  C  8   C

#define CLK_LEN 400             // clock pulse (us)
#define SERVO_MIN 630           // servo minimum value (us); operates fine in testing down to 500us
#define SERVO_MAX 1590          // servo maximum value (us)
#define OPIN 4                 // output pin
#define PRESCALE 80             // timer prescaler 80/80M = 1us cycle time to give flexible timings to compensate for code overhead
#define FRONT_PORCH 5000        // mandatory low period prior to data packet (5ms)

volatile uint64_t servo[8]={SERVO_MIN,SERVO_MIN,SERVO_MIN,SERVO_MIN,SERVO_MIN,SERVO_MIN,SERVO_MIN,SERVO_MIN};       // default servo to min value, used by ISR

volatile uint32_t state=0;          // output is driven by a simple state machine, managed inside ISR
volatile uint32_t ch_ctr=0;         // channel counter for the output, managed inside ISR
volatile uint32_t framectr=0;      // count how many frames have been sent (not required used for debug)
volatile uint64_t nextus;           // used for calculate the time overhead of the ISR code and shorten the next interval accordingly

hw_timer_t *Timer1_Cfg = NULL;

portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR Timer1_ISR()
{
    portENTER_CRITICAL_ISR(&timerMux);

    digitalWrite(OPIN, !digitalRead(OPIN));   // toggle output pin here for fastest responnse
    timerStop(Timer1_Cfg);                    // stop the timer until the alarm is ready to be set again
    
    switch (state) {              // state machine, state is current position in data packet
      case 0:               // 0 = front porch/idle end, IRQ fires when front porch ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 1:               // 1 = First clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 1 servo gap
        break;
      case 2:               // 2 = channel 1 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 3:               // 3 = #2 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 2 servo gap
        break;
      case 4:               // 4 = channel 2 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 5:               // 5 = #3 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 3 servo gap
        break;
      case 6:               // 6 = channel 3 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 7:               // 7 = #4 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 4 servo gap
        break;
      case 8:               // 8 = channel 4 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 9:               // 9 = #5 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 5 servo gap
        break;
      case 10:              // 10 = channel 5 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 11:              // 11 = #6 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 6 servo gap
        break;
      case 12:              // 12 = channel 6 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 13:              // 13 = #7 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 7 servo gap
        break;
      case 14:              // 14 = channel 7 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 15:              // 15 = #8 clock pulse end, fires when clock pulse ends
        state++;
        nextus=servo[ch_ctr++]; // this is channel 8 servo gap
        break;
      case 16:              // 16 = channel 8 servo gap end, IRQ fires when servo gap ends
        state++;            
        nextus=CLK_LEN;     // this is a clock pulse
        break;
      case 17:              // 17 = #9 clock pulse end, fires when clock pulse ends
        state++;
        nextus=FRONT_PORCH; // end of data packet, send front porch
        state=0;            // reset state to front porch
        ch_ctr=0;           // reset channel counter
        framectr++;         // increment the sent frame counter (not required, only used for debug)
        digitalWrite(OPIN, 0);    // ensure the output is low for the front porch
        break;
      default:              // should never hit here, but if it does, reset the state machine
        nextus=FRONT_PORCH; // end of data packet, send front porch
        state=0;            // reset state to front porch
        ch_ctr=0;           // reset channel counter
        digitalWrite(OPIN, 0);    // ensure the output is low for the front porch
        break;
      }

    timerAlarmWrite(Timer1_Cfg, nextus, false);           // set the alarm value for this pulse
    timerAlarmEnable(Timer1_Cfg);                         // enable the alarm interrupt
    timerWrite(Timer1_Cfg,0);                             // reset the timer for proper alarm value compare
    timerStart(Timer1_Cfg);                               // start the timer again

    portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {
  
  analogReadResolution(10);                     // Read the pots with 0-1024 values 

  pinMode(OPIN, OUTPUT);
  digitalWrite(OPIN, 0);                        // default output state is low
  Timer1_Cfg = timerBegin(0, PRESCALE, true);
  
  timerAttachInterrupt(Timer1_Cfg, &Timer1_ISR, true);
  timerAlarmWrite(Timer1_Cfg, FRONT_PORCH, false);      // start with a 5000us dead zone
  timerWrite(Timer1_Cfg,0);
  timerAlarmEnable(Timer1_Cfg);                         // start the timer
}

void loop() {
  
    int pot1,pot2,pot3;

  pot1=SERVO_MAX-analogRead(33);    
  //pot1=SERVO_MIN+analogRead(33);      // alternate if reverse knob rotation is desired
  pot2=SERVO_MAX-analogRead(34);
  //pot2=SERVO_MIN+analogRead(34);      // alternate if reverse knob rotation is desired
  pot3=SERVO_MAX-analogRead(35);
  //pot3=SERVO_MIN+analogRead(35);      // alternate if reverse knob rotation is desired

  // check that the servo values stay in bounds
  servo[1]=(pot2<SERVO_MIN) ? pot2: SERVO_MIN;
  servo[1]=(pot2<SERVO_MAX) ? pot2: SERVO_MAX;
  servo[2]=(pot3<SERVO_MIN) ? pot3: SERVO_MIN;
  servo[2]=(pot3<SERVO_MAX) ? pot3: SERVO_MAX;
  servo[3]=(pot1<SERVO_MIN) ? pot1: SERVO_MIN;
  servo[3]=(pot1<SERVO_MAX) ? pot1: SERVO_MAX;

  delay(10);      // delay is asthetic...not required
}
