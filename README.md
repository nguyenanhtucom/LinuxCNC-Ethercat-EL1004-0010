1. Create file lcec_el1004_0010.c(copy from file lcec_elxxx.c)
    
    Relace funtion name "lcec_el1xxx" to "lcec_el1004_0010"
    
    Edit ...
    

  for (i=0, pin=hal_data; i<slave->pdo_entry_count; i++, pin++) {
    // initialize POD entry

    LCEC_PDO_INIT(pdo_entry_regs, slave->index, slave->vid, slave->pid, 0x3101, 0x01 + i, &pin->pdo_os, &pin->pdo_bp);

    // export pins
    if ((err = lcec_pin_newf_list(pin, slave_pins, LCEC_MODULE_NAME, master->name, slave->name, i)) != 0) {
      return err;
    }
  }

2. Create file lcec_el1004_0010.h(copy from file lcec_elxxx.h)

    #ifndef _LCEC_EL1004_0010_H_
    #define _LCEC_EL1004_0010_H_

    #include "lcec.h"

    #define LCEC_EL1004_0010_VID LCEC_BECKHOFF_VID

    #define LCEC_EL1004_0010_PID 0x03EC3052

    #define LCEC_EL1004_0010_PDOS 4
    
    int lcec_el1004_0010_init(int comp_id, struct lcec_slave *slave, ec_pdo_entry_reg_t *pdo_entry_regs);

    #endif

    
3. Add "lcec_el1004_0010.o \"  in file Kbuild

4. Edit file lcec_conf.h 

  Add "lcecSlaveTypeEL1004_0010" in typedef enum 

  typedef enum {
  lcecSlaveTypeEL1004_0010,
  lcecSlaveTypeInvalid,
  lcecSlaveTypeGeneric,
  . . .
 
5. Edit file lcec_conf.c 

  Add "{ "EL1004-0010", lcecSlaveTypeEL1004_0010, NULL }," in array slaveTypes[]
 
  static const LCEC_CONF_TYPELIST_T slaveTypes[] = {
  // bus coupler
    { "EL1004-0010", lcecSlaveTypeEL1004_0010, NULL },
    { "EK1100", lcecSlaveTypeEK1100, NULL },
    . . .
  6. Edit lcec_main.c
    Add #include "lcec_el1004_0010.h"
    
    Add to array types[] value: { lcecSlaveTypeEL1004_0010, LCEC_EL1004_0010_VID, LCEC_EL1004_0010_PID, LCEC_EL1004_0010_PDOS, lcec_el1004_0010_init},
    
    static const lcec_typelist_t types[] = {
  // bus coupler
    { lcecSlaveTypeEK1100, LCEC_EK1100_VID, LCEC_EK1100_PID, LCEC_EK1100_PDOS, NULL},
    { lcecSlaveTypeEL1004_0010, LCEC_EL1004_0010_VID, LCEC_EL1004_0010_PID, LCEC_EL1004_0010_PDOS, lcec_el1004_0010_init},
     . . .
     
  # Test
  1. ethercat-conf.xml
   <masters>  
  <master idx="0" appTimePeriod="1000000" refClockSyncCycles="1000">
    <slave idx="0" type="EK1100" name="D1"/>
    <slave idx="1" type="EL2042" name="D2"/>
    <slave idx="2" type="EL1004-0010" name="D3"/>
  </master>
</masters>
$ halcmd:
loadrt trivkins
loadrt motmod servo_period_nsec=1000000 num_joints=3
loadusr -W lcec_conf /home/nat/Desktop/ethercat-conf.xml
loadrt lcec
addf lcec.read-all servo-thread
addf lcec.write-all servo-thread
