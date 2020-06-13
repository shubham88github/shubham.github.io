---
title: "Status LED Kernel Module"
excerpt_separator: "<!--more-->"
categories:
  - kernel
tags:
  - c
  - kernel module
  - linux
  - character device driver
---

I have worked with GPS/GSM/4G CANBus Emulators/Simulators. Used python/bash/C/C++ to interface these modules. And everytime they come with drivers. Sometimes these drivers have problems, and id have to get the manufacturers help in order get the problem resolved usually. I was always curious on how these drivers worked. So here was my go at it.

I have a raspberry pi, and i had an RGB led attached to it. I wanted this to be the status led. And to make things interesting i wanted to control the leds via a kernel module. I wanted programs to send commands to the kernel module via the character device file. Which would control how the leds would blink.

### Commands
The commands have 3 numbers in it.
1. The color
  * RED   3
  * GREEN 4
  * BlUE  5
2. Period of time for the led to blink
  * SHORT   6
  * NORMAL  7
  * LONG    8
3. Number of times the led should blink
  * It should be between 1 to 9 blinks.

### Example Command
* 3 8 2
  * Blink the red led with a long delay 2 times.
* 5 7 7
  * Blink the blue led with a normal delay 7 times.
* 4 8 1
  * Blink the green led with a long delay once.

### Code
Since the requirements have already been defined, we will jump into the code design. I came up with a very simple solution. I will have a linked list which takes list of all the led state tasks which needs to be carried out. And with a timer interrupt it will interrupt based on the task list. So if you issue the command which relates to bink the red led 2 times with a long period. It will add 4 tasks :

#### Tasks
1. Turn red led on for a long period.
2. Turn red led off for a long period.
3. Turn red led on for a long period.
4. Turn red led off for a long period.

#### Handling the character device file
```c
/**
 * @file    chardev.h
 * @author  Eshan Shafeeq
 * @version 0.1
 * @date    29 March 2017
 * @brief   This file provides the device driver
 *          capabilities to the module including
 *          this file. Fetches a major number.
 *          Registers a device class and a device.
 *          Also implements open, release, read and write
 *          functionality for the file operations structure.
 *
 **/

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <asm/uaccess.h>
#include <linux/device.h>
#include "printops.h"
#include "command_process.h"

#ifndef _CHARDEV_H_
#define _CHARDEV_H_


#define DEVICE_NAME     "sled"
#define DEVICE_CLASS    "sled_class"


/**
 * Module Global Variables
 * -----------------------
 *  @param  major_number        : To store the major number assigned
 *                                  to this device;
 *  @param  cmd_buff            : A buffer to store the command from
 *                                  the user.
 *  @param  cmd_buff_len        : To store the length of the buffer
 *                                  recieved from the user.
 *  @param  cmd_device_class    : To store the device class structure.
 *  @param  cmd_device          : To store the registered device structure.
 *
 **/

static int      major_number;
//static char     cmd_buff;
//static short    cmd_buff_length;

static struct   class*  cmd_device_class    = NULL;
static struct   device* cmd_device          = NULL;

/**
 * Prototype Functions [FOPS]
 * --------------------------
 **/

static int      device_open(    struct inode *,
                                struct file * );
static int      device_release( struct inode *,
                                struct file * );
static ssize_t  device_read(    struct file *,
                                char   *,
                                size_t,
                                loff_t * );
static ssize_t  device_write(   struct file *,
                                const char *,
                                size_t,
                                loff_t * );

/**
 * File Operations Structure
 * -------------------------
 **/
static struct file_operations fops = {
    .open       =   device_open,
    .release    =   device_release,
    .read       =   device_read,
    .write      =   device_write
};

/**
 * Char device Open
 * ----------------
 **/
static int device_open( struct inode *ptr_inode, struct file *ptr_file ){
    kern_info( 0, "Device has been opened");
    return 0;
}

/**
 * Char device Release
 * -------------------
 **/
static int device_release( struct inode *ptr_inode, struct file *ptr_file ){
    kern_info( 0, "Device has been released");
    return 0;
}

/**
 * Char device Read
 * ----------------
 **/
static ssize_t device_read( struct file *ptr_file, char *buff,
                            size_t buff_len, loff_t *offset ){
    return 0;
}

/**
 * Char device Write
 * -----------------
 **/
static ssize_t device_write( struct file *ptr_file, const char *buff,
                             size_t buff_len,       loff_t *offset ){
    kern_info( 0, "Recieved %d bytes from user", buff_len );
    process_command( buff, buff_len );

    return buff_len;
}


/**
 * Setup character device function
 * -------------------------------
 **/
static int setup_chardev(void){
    kern_info( 0, "Character device setup");

    //Major number setup
    major_number = register_chrdev( 0, DEVICE_NAME, &fops );
    if( major_number < 0 ){
        kern_alert( 0, "Failed to obtain a MAJOR_NUMBER");
        return major_number;
    }

    kern_info( 3, "Successfully Obtained the MAJOR_NUMBER : %d", major_number );

    //Device class setup
    cmd_device_class = class_create( THIS_MODULE, DEVICE_CLASS );
    if( IS_ERR( cmd_device_class ) ){
        unregister_chrdev( major_number, DEVICE_NAME );
        kern_alert( 0, "Failed to register the DEVICE CLASS");
        return PTR_ERR( cmd_device_class );
    }

    kern_info( 0, "Successfully Registered the DEIVCE CLASS");

    //Deivce Registration setup
    cmd_device = device_create( cmd_device_class, NULL, MKDEV( major_number, 0 ),
                                NULL,                              DEVICE_NAME );
    if( IS_ERR( cmd_device ) ){
        class_destroy( cmd_device_class );
        unregister_chrdev( major_number, DEVICE_NAME );
        kern_alert( 0, "Failed to register device");
        return PTR_ERR( cmd_device );
    }

    kern_info( 0, "Successfully Registered the DEVICE");

    return 0;
}

/**
 * Remove the character device functionality
 * -----------------------------------------
 **/
static void remove_chardev(void){

    kern_info( 0, "Character device removal");

    //Remove the device
    device_destroy( cmd_device_class, MKDEV( major_number, 0) );

    //Remove class
    class_unregister( cmd_device_class );
    class_destroy( cmd_device_class );

    //Unregbister Major number
    unregister_chrdev( major_number, DEVICE_NAME );
}
#endif
```
#### Handle the GPIO pins
After taking care of the character device file, we need code to actually turn on and turn off the GPIOs connected to the led. Ill be using GPIO pins 4, 17, 27 on the raspberry pi in this example.

```c
/**
 * @file    led_gpio.h
 * @author  Eshan Shafeeq
 * @version 0.1
 * @date    31 March 2017
 * @brief   This file is to give access to
 *          gpio pins to control leds.
 **/

#include <linux/gpio.h>
#include "printops.h"

#ifndef _LED_GPIO_H_
#define _LED_GPIO_H_

/**
 * Structure to hold the array of
 * leds with initial state
 **/

static struct gpio leds[] = {
    {   4,  GPIOF_OUT_INIT_HIGH, "REDLED"    },
    {   17, GPIOF_OUT_INIT_HIGH, "GREENLED"  },
    {   27, GPIOF_OUT_INIT_HIGH, "BLUELED"   }
};

/**
 * To know whether the gpio's
 * have been assigned
 **/

static bool initialized;

/**
 * Initialization function
 *
 * @brief   Requests for the required
 *          array of gpios.
 * @param   ret     To store the state of
 *                  the request function.
 **/

static inline void initiate_leds( void ){
    int ret =0;

    ret = gpio_request_array( leds, ARRAY_SIZE( leds ) );
    if( ret < 0 ){
        kern_alert( 0, "Failed to initialize leds" );
        initialized = false;
        return;
    }

    kern_info( 0, "Leds have been initialised Succesfully" );
    initialized = true;

}

/**
 * Destructor
 *
 * @brief   Release the gpio pins which
 *          was requested previously.
 **/

static inline void release_leds( void ){
    size_t i;

    for( i=0; i<ARRAY_SIZE( leds ); i++ ){
        gpio_set_value( leds[i].gpio, 0 );
    }

    gpio_free_array( leds, ARRAY_SIZE( leds ) );
    kern_info( 0, "Leds have been released" );
    initialized = false;
}

/**
 * Toggle led
 *
 * @brief   This function sets a given led on or off
 *
 * @param   index   The index of the led to be toggled
 * @param   value   The boolean state that the led should have.
 *
 **/

static inline void toggle_led( size_t index, bool value ){

    //kern_info( 5, "index %d state %s", index, value?"true":"false" );
    if( value ){
        gpio_set_value( leds[ index ].gpio, 0 );
    }else{
        gpio_set_value( leds[ index ].gpio, 1 );
    }
}

#endif
```
#### Process the commands

The commands which have been passed into the character device file needs to be processed accordingly.
```c
/**
 * @file    process_command.h
 * @author  Eshan Shafeeq
 * @version 0.1
 * @date    31 March 2017
 * @brief   This file is to process commands received
 *          from the user through character device file
 *          and assign the appropriate function based on
 *          the command parameters.
 **/


#include <linux/kernel.h>
#include <linux/module.h>
#include "printops.h"
#include "interrupt.h"

#ifndef _PROCESSCOMMAND_H_
#define _PROCESSCOMMAND_H_

/**
 * Validate buffer
 *
 * @brief   This function validates the buffer received
 *          from the user. Makes sure only white space
 *          and numeric values are given as command.
 *
 * @param   buff        The buffer received from the user
 * @param   buff_len    The length of the buffer
 * @param   buff_i      The value of buff[i] converted to
 *                      ACII value.
 * @param   space       To keep count of the white space
 * @param   num         To keep count of the numeric digits
 *
 **/
static inline bool validate_buffer( const char *buff, size_t buff_len ){
    size_t i;
    int buff_i = 0;
    short space=0;
    short num=0;

    for( i=0; i<buff_len-1; i++ ){
        buff_i = buff[i] - '0';
        //debug
        //kern_info( 0, "processing #%i : %c - %d", i, buff[i], buff_i );
        //
        if( buff_i == -16 || buff_i > -1 ){
            if( buff_i == -16 )
                space++;
            if( buff_i > -1 && buff_i < 10 )
                num++;
        }else{
            return false;
        }
    }
    if( space == 2 && num == 3 )
        return true;
    else
        return false;
}

/**
 * Process Command
 *
 * @brief   This function processes the command
 *          received by the buffer and executes
 *          the proper action.
 *
 * @param   buff        The buffer received from the user
 * @param   buff_len    The length of the buffer
 * @param   color       To store the color selected by the user
 * @param   delay       To store the delay length selected by the user
 * @param   qty         To store the number of blinks selected
 *
 **/
static void process_command( const char *buff, size_t buff_len ){
//    size_t i;
    short color;
    short delay;
    short qty;

    if( validate_buffer( buff, buff_len ) ){
        kern_info( buff_len-2, "buffer : %s", buff);
        /*  --DEBUG--
         *  kern_info(0, "     index: char|ascii|int cast");
            kern_info(0, "     --------------------------");
        for( i=0; i<buff_len-1; i++ ){
            kern_info(0, "analysis #%i:  %c | %d  | %d", i, buff[i], buff[i], buff[i] - '0');
        }*/

        color = buff[0] - '0';
        delay = buff[2] - '0';
        qty   = buff[4] - '0';

        switch(color){
            case RED:
                kern_info( 0, "RED color selected");
                break;
            case GREEN:
                kern_info( 0, "GREEN color selected");
                break;
            case BLUE:
                kern_info( 0, "BLUE color selected");
                break;
        }
        switch(delay){
            case SHORT:
                kern_info( 0, "SHORT time delay selected");
                break;
            case NORMAL:
                kern_info( 0, "NORMAL time delay selected");
                break;
            case LONG:
                kern_info( 0, "LONG time delay selected");
                break;
        }

        kern_info( 0, "QTY : %d", qty );
        start_timer_interrupt(color, delay, qty);

    }

}

#endif
```
#### Handle the timer interrupt routines

In the following file i have defined the structures used to for the linkedlist, which helps with the command processing in a stack like manner. To prevent multiple timer interrupts i used a semaphore to restrict access. This code is also where all the tasks get added to the list. All the interrupt routines are here as well.
```c
/**
 * @file    interrupt.h
 * @author  Eshan Shafeeq
 * @version 0.1
 * @date    31 March 2017
 * @brief   This file implements the timer
 *          interrupt function to blink an
 *          led the requested amount of times
 *          concurrently. It uses a semaphore
 *          to prevent crashes.
 *
 **/
#include <linux/timer.h>
#include <linux/sched.h>
#include <linux/semaphore.h>
#include <linux/list.h>
#include <linux/slab.h>
#include "printops.h"
#include "led_gpio.h"

#ifndef _INTERRUPT_H_
#define _INTERRUPT_H_

#define     DELAY_TIME      50
#define     SHORT_DELAY     10
#define     NORMAL_DELAY    50
#define     LONG_DELAY      100

#define     RED             3
#define     GREEN           4
#define     BLUE            5

#define     SHORT           6
#define     NORMAL          7
#define     LONG            8

/**
 * Globals
 *
 * @param   interrupt   The timer_list structure to
 *                      interrupt regularly.
 * @param   running     The semaphore to prevent access
 *                      to the timer start function.
 * @param   state_list  The structure to hold the led
 *                      states over time.
 * @param   state_len   To store the size of the state
 *                      list.
 *
 **/

static struct timer_list interrupt;
static struct semaphore running;

struct led_state {
    int     id;
    short   color;
    short   delay;
    bool    state;

    struct list_head head;
};
struct led_state state_list;
static int state_len;

/**
 * Create new Task
 *
 * @brief   To dynamically allocate space for
 *          a new led_state object and return
 *          the address to the caller.
 *
 * @param   i           Id of the newly created object
 * @param   c           Color of the newly created object
 * @param   d           Delay time of newly created object
 * @param   s           State of the newly created object
 * @param   new_state   The pointer to access the newly created object
 *
 **/
static struct led_state * create_new_task(int i, short c, short d, bool s){

    struct led_state *new_state;

    new_state = kmalloc( sizeof( *new_state ), GFP_KERNEL );
    new_state->id       = i;
    new_state->color    = c;

    switch( d ){

        case SHORT:
            new_state->delay    = SHORT_DELAY;
            break;
        case NORMAL:
            new_state->delay    = NORMAL_DELAY;
            break;
        case LONG:
            new_state->delay    = LONG_DELAY;
            break;

    }
    new_state->state    = s;

    INIT_LIST_HEAD( &new_state->head );
    state_len++;

    return new_state;

}

/**
 * Get Delay
 *
 * @brief   obtain the delay amount from the
 *          object recieved via the value index.
 *
 * @param   value   index of the state_list object which
 *                  the delay time should be obtained from.
 *
 **/

static short get_delay( unsigned long value ){
    struct led_state *state;
    list_for_each_entry( state, &(state_list.head), head ){
        if( value==state->id ){
            return state->delay;
        }
    }
    return DELAY_TIME;
}

/**
 * Create a task list
 *
 * @brief       Create a linkedlist of led_state type objects.
 *
 * @param   color       Color variable of all the objects in the list.
 * @param   delay       Delay time for all the objects
 * @param   qty         Number of blinks for the specific list.
 * @param   new_state   The pointer to hold a reference to newly
 *                      created object.
 *
 **/
static void create_task_list( short color, short delay, short qty ){

    struct led_state *new_state;
    size_t i;

    kern_info( 0, "Initializing task list" );

    INIT_LIST_HEAD( &state_list.head );
    state_len=0;

    //Populate the list
    for( i=0; i<(qty*2); i+=2 ){
        new_state = create_new_task(i, color, delay, true);
        list_add_tail( &(new_state->head), &(state_list.head) );

        new_state = create_new_task(i+1, color, delay, false);
        list_add_tail( &(new_state->head), &(state_list.head) );
    }


}

/**
 * Destroy Task List
 *
 * @brief   Release the memory dynamically allocated
 *          to the state list once it is not required.
 *
 * @param   state_iter      A pointer variable to iterate the list
 * @param   state_iter_tmp  A pointer variable for safe iteration
 *
 **/

static void destroy_task_list( void ){
    struct led_state *state_iter;
    struct led_state *state_iter_tmp;

    list_for_each_entry_safe( state_iter, state_iter_tmp, &(state_list.head), head ){
        list_del( &(state_iter->head) );
        kfree( state_iter );
    }
}

/**
 * Trigger Led
 *
 * @brief   This function triggers the selected
 *          led based on the number of blinks.
 *
 * @param   state   A pointer to obtain the values
 *                  from the chosen state.
 * @param   value   The current index of the chosen
 *                  object from the linked list.
 *
 **/
static void trigger_led( unsigned long value ){
    struct led_state *state;

    list_for_each_entry( state, &(state_list.head), head ){
        if( value==state->id ){
            //kern_info( 0, "color : %hu", state->color );
            //kern_info( 0, "delay : %hu", state->delay );
            //kern_info( 0, "state : %hu", state->state );
            switch( state->color ){
                case RED:
                    toggle_led( 0, state->state );
                    break;
                case GREEN:
                    toggle_led( 1, state->state );
                    break;
                case BLUE:
                    toggle_led( 2, state->state );
                    break;

            }
        }
    }
}

/**
 * Interrupt Routine
 *
 * @brief   This is the interrupt routine which
 *          happen based on the delay time
 *          requested by the user.
 *
 * @param   value   The count of the current iteration.
 *                  also to keep track of the current
 *                  state from the state list.
 *
 **/
static void interrupt_routine( unsigned long value ){

    //kern_info( 0, "routine %ld", value );
    trigger_led( value );
    interrupt.data = ++value;
    interrupt.expires = jiffies + get_delay( value );
    if( value < state_len ){
        add_timer( &interrupt );
    }else{
        destroy_task_list();
        release_leds();
        up( &running );
        kern_info( 0, "Timer stopped");
    }

}

/**
 * State Timer Interrupt
 *
 * @brief   The function which starts all the magic
 *
 * @param   color   The color obtained from the command.
 * @param   delay   The delay time obtained from the command.
 * @param   qty     The blink amount obtained from the command.
 *
 **/

static bool start_timer_interrupt( short color, short delay, short qty ){
    kern_info( 0, "Timer started");
    if( down_interruptible(&running) == 0 ){
        interrupt.data = 0;

        create_task_list( color, delay, qty );
        initiate_leds();

        interrupt.expires = jiffies + DELAY_TIME;
        add_timer( &interrupt );
        return true;
    }else{
        return false;
    }

}
/**
 *  Setup Timer Interrupts
 *
 *  @brief  Initiates the semaphore and the interrupt function.
 *
 **/
static void setup_timer_interrupt(void){

    kern_info( 0, "Setting up timer interrupt" );
    sema_init( &running, 1 );
    init_timer( &interrupt );
    interrupt.function= interrupt_routine;
}

/**
 *  Remove Timer
 *
 *  @brief  Function to end all the magic happening.
 *
 **/
static void remove_timer(void){

    kern_info( 0, "Removing timer interrupt");

    del_timer_sync( &interrupt );
}
#endif
```

#### Misc code
As youve noticed, ive been using this function called `kern_info` since i wanted to avoid typing the long original kernel logging statement `printk`. Here is the file which defines all these functions. Ill be using these in all my kernel modules i write in the future.

```c
/**
 * @file    printops.h
 * @author  Eshan Shafeeq
 * @version 0.1
 * @date    25 March 2017
 * @brief   This file contains some easy functions
 *          to print out messages to the kernel log
 **/

#include <linux/kernel.h>
#include <linux/module.h>
#ifndef _PRINTOPS_H_
#define _PRINTOPS_H_

/**
 * kern_info
 *
 * @brief   Prints a message to the kernel buffer
 *          with the message priority of KERN_INFO.
 *          Usually for debugging purposes.
 *
 **/

static void kern_info(  int len, const char *format, ...){
    char    msg[256];
    va_list args;

    va_start( args, format );
        vsnprintf(msg, strlen( format ) + len + 1, format, args );
    va_end( args );
    printk(KERN_INFO "[SLED]: %s\n", msg);
}
/**
 * kern_alert
 *
 * @brief   Prints a message to the kernel buffer
 *          with a higher priority. Usually for
 *          alert messages
 *
 **/

static void kern_alert(  int len, const char *format, ...){
    char    msg[256];
    va_list args;

    va_start( args, format );
        vsnprintf(msg, strlen( format ) + len + 1, format, args );
    va_end( args );
    printk(KERN_ALERT "[SLED]: %s\n", msg);
}
#endif
```

The main file which defines the all the necessary definitions to make a kernel module.
Also this file defines the `__init` and the `__exit` methods for the kernel module.

```c
/**
* @file    chardev_command.c
* @author  Eshan Shafeeq
* @date    30 March 2017
* @version 0.1
* @brief   A character device driver capable
*          of receiving commands and distinguishing
*          them and giving outputs based on the
*          commands received.
**/

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>
#include "chardev.h"
#include "interrupt.h"

/**
* Module definitions
* ------------------
**/
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Eshan Shafeeq");
MODULE_DESCRIPTION("Device driver to recieve commands and execute them");
MODULE_VERSION("0.1");
MODULE_SUPPORTED_DEVICE(DEVICE_NAME);

/**
* Module Initialization
* ---------------------
**/
static int __init cmd_dev_init(void){
   int ret;

   ret = setup_chardev();
   if( ret != 0 ){
       return ret;
   }

   setup_timer_interrupt();

   return 0;
}

/**
* Module Termination
* ------------------
**/
static void __exit cmd_dev_exit(void){
   remove_chardev();      
   remove_timer();
}

module_init( cmd_dev_init );
module_exit( cmd_dev_exit );
```

And finally the humble makefile i used to build the driver

```makefile

obj-m +=sled.o
sled-objs := start.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

So if you were to try out this kernel module on your raspberry pi 3. Here are the commands you need to issue to insert the kernel module into the kernel and then give the appropriate permission for the device file, so that you/your programs can wirte your commands to the device file.

```shell
# After you have built the module into the sled.ko
# You need to load it into the kernel
$ sudo insmod sled.ko
# Very important to check if the module has been loaded and you can see the kernel log using
$ dmesg
# If you have verified that the kernel module has been loaded, now you have to set appropriate permissions on the device file.
$ sudo chmod 666 /dev/sled
# Now you can start sending commands to the device file and watch the leds blink.
$ echo "3 8 3" > /dev/sled
```
Hope this guide helps someone out there!
And keep shooting for the stars!
