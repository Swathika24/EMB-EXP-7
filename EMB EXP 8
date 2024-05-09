#include <LPC17xx.H>
#include "GLCD.h"
#include "Serial.h"

#define __FI 1

unsigned int Channel_No;
unsigned char result,Digital;

	
/* To Display the Channel number in the ASCII format */
void Decimal_Adjust(unsigned long Channel_No)
{
  switch(Channel_No)
  {
  case 0xa:	SER_PutChar('0');
		GLCD_DisplayChar(6,12, __FI,'0');
				break;
					 
	case 0xb:	SER_PutChar('1');
			GLCD_DisplayChar(6,12, __FI,'1');
				break;
					 
	case 0xc:	SER_PutChar('2');
			GLCD_DisplayChar(6,12, __FI,'2');
				break; 

	case 0xd:	SER_PutChar('3');
			GLCD_DisplayChar(6,12, __FI,'3');
				break; 

	case 0xe:	SER_PutChar('4');
			GLCD_DisplayChar(6,12, __FI,'4');
				break; 

	case 0xf:	SER_PutChar('5');
			GLCD_DisplayChar(6,12, __FI,'5');
				break; 

  }
}

/* Routine to convert the Analog Data  */
void Convert(void)				    	
{
  unsigned long temp = 0,i = 0,temp1 = 0,res;	
  temp1 = (Channel_No << 15);
  LPC_GPIO0->FIOPIN = (LPC_GPIO0->FIOPIN & 0xFF807FFF) |  temp1;
  LPC_GPIO0->FIOPIN = (LPC_GPIO0->FIOPIN & 0xFF807FFF) | ((1 << 20) | temp1);     /* Assert the START signal */
  
  temp1 = 0;
  LPC_GPIO0->FIOPIN = ((LPC_GPIO0->FIOPIN & 0xFF807FFF) | (Channel_No | temp1));	 /* Pull down the START signal */
	
  for(i = 0; i <= 10000; i ++);
  temp = LPC_GPIO0->FIOPIN &  0x00800000;	
  temp >>= 23;
  temp = temp & 0x01;	 				/* Wait for the End of Conversion */
  while(temp != 0x1)
  { 
   	temp = LPC_GPIO0->FIOPIN &  0x00800000;					
	  temp >>= 23;
  	temp = temp & 0x01;	
  } 
 
  LPC_GPIO0->FIOPIN = ((LPC_GPIO0->FIOPIN & 0x007F8000) | ((1<<21) | Channel_No));
    for(i = 0; i <= 10000; i ++);
	
  temp = LPC_GPIO1->FIOPIN &  0x07F80000;		
  temp >>= 19;
  										/* Read the Digital Value */ 
  temp1 = 0;
  LPC_GPIO0->FIOPIN = ((LPC_GPIO0->FIOPIN & 0x007F8000) | (Channel_No | temp1)); 
  res = LPC_GPIO1->FIOPIN &  0x07F80000;	
  res >>= 19;
  result = (unsigned char)res;			  		
}

void Display(void)
{
  unsigned char temp = 0;
  SER_SendString("\nChannel Number : ");		
  GLCD_DisplayString(6, 0, __FI,"Channel No=");		
  if(Channel_No <= 9)					/* Check whether the channel no is < than 9 */
  {
  	SER_PutChar('0');
	  SER_SendHex(Channel_No);

	GLCD_DisplayChar(6,11, __FI, '0');
		GLCD_DisplayChar(6,12, __FI, '0');
		GLCD_DisplayChar(6,12, __FI,Channel_No+ 0x30);

  }
  else if((Channel_No >= 0x0a) && (Channel_No <= 0x0f))
  {
   	SER_PutChar('1');
	GLCD_DisplayChar(6,11, __FI, '1');
  }
	                            			/* If > 9, send the ASCII values depending  */
  	Decimal_Adjust(Channel_No);			/* upon the value of the channel number */
  	SER_SendString("\tDigital Value : ");                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                
  	GLCD_DisplayString(7, 0, __FI,"Digital Value=0x");                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                
    temp = (result & 0xf0) >> 4;
  	SER_SendHex(temp);
    temp = (result & 0xf0) >> 4;
			if( temp <= 9)					/* Displays the corresponding ASCII value of 2nd digit */
   		 	temp = temp + 0x30;
  		else
   			temp = temp + 0x37;
	  
  	GLCD_DisplayChar(7,16, __FI,temp);

	  SER_SendHex(result & 0x0f);
			temp=(result & 0x0f);
			if(  temp <= 9)					/* Displays the corresponding ASCII value of 2nd digit */
   		 	temp = temp + 0x30;
  		else
   			temp = temp + 0x37;
	  
	  GLCD_DisplayChar(7,17, __FI,temp);


    SER_SendString("\n\r");
}


void Disp_Update(void)					    /* Routine to update the Channel numbers */
{
  		      char c;
  		c = SER_GetChar();          /* Read a character from UART */
  
	if(c == ',')
  	{
    if(Channel_No == 15)					/* If typed ',',increment the channel number */
		   Channel_No = 0;
	  else 
    	 Channel_No += 1;
    }											         
 else if(c == '-')              /* If typed '-',decrement the channel number */
  {
   if(Channel_No == 0)
		  Channel_No = 15;
	 else
		  Channel_No  -= 1;
  }
}
main()
{
	  LPC_SC->PCONP     |= (1 << 15);            /* enable power to GPIO    */

  LPC_GPIO0->FIODIR |=   0x007F8000;	         /* PortA output PortC input */
  LPC_GPIO1->FIODIR &= ~(0x07F80000);          /* PortB input */
  SER_Init();
	
  #ifdef __USE_LCD
  GLCD_Init();                               /* Initialize graphical LCD      */
  GLCD_Clear(White);                         /* Clear graphical LCD display   */
  GLCD_SetBackColor(Blue);
  GLCD_SetTextColor(White);
  GLCD_DisplayString(0, 0, __FI, "        ESA         ");
  GLCD_DisplayString(1, 0, __FI, "     Bangalore      ");
  GLCD_DisplayString(2, 0, __FI, "  www.esaindia.com  ");
  GLCD_SetBackColor(White);
  GLCD_SetTextColor(Blue);
  GLCD_DisplayString(5, 0, __FI, "   16 Channel ADC");
	 
#endif
 Channel_No = 0;
  
  while(1)
  {	
		
 	Convert();							/* Convert the analog to digital value */
	Display();							/* Display the correspoding Channel number */
										      /* and it's equivalent digital value */
 	Disp_Update();					/* Update the display and channel number */			   
  }	
}
