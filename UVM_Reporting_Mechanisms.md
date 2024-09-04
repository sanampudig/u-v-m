# Universal Verification Methodology (UVM)

## Reporting Mechanisms

- **UVM_INFO**: Informative message.
- **UVM_WARNING**: Indicates a potential problem.
- **UVM_ERROR**: Indicates a real problem. Simulation continues subject to the configured message action.
- **UVM_FATAL**: Indicates a problem from which simulation cannot recover. Simulation exits via `$finish` after `#0`.

---

## Syntax for UVM Information Reporting

```systemverilog
`uvm_info(ID, MSG, Verbosity Level)
`uvm_warning(ID, MSG)
`uvm_error(ID, MSG)
`uvm_fatal(ID, MSG)
```

### Verbosity Levels
- `UVM_NONE = 0`
- `UVM_LOW = 100`
- `UVM_MEDIUM = 200`
- `UVM_HIGH = 300`
- `UVM_FULL = 400`
- `UVM_DEBUG = 500`

The **default verbosity level** is 200. Messages with a verbosity level of 200 or lower will appear on the console.

---

### Including UVM Macros

```systemverilog
`include "uvm_macros.svh"  // Includes macros like `uvm_info`
import uvm_pkg::*;
```

---

### Example Code

```systemverilog
module tb;
  initial begin
    #50;
    `uvm_info("TB_TOP", "Hello World", UVM_LOW);
    $display("Hello World with Display");
  end
endmodule
```

**Output:**

```
KERNEL: UVM_INFO /home/runner/testbench.sv(9) @ 0: reporter [TB_TOP] Hello World
KERNEL: Hello World with Display
```

---

### Printing Variables with UVM

To print a variable, use `$sformatf`.

```systemverilog
int data = 56;

initial begin
  `uvm_info("TB_TOP", $sformatf("Value of data: %0d", data), UVM_NONE);
end
```

### Displaying Messages on the Console

For a message to appear on the console, its verbosity level must be lower than or equal to the set verbosity level.

- Example: If you set the verbosity level to `UVM_HIGH`, the message will be displayed on the console only if the set verbosity is 300 or higher (UVM_HIGH, UVM_FULL, UVM_DEBUG). Medium, low, or none cannot print this message.

```systemverilog
$display("Default Verbosity level: %0d ", uvm_top.get_report_verbosity_level);
uvm_top.set_report_verbosity_level(UVM_HIGH);
```

You can also change the verbosity level from the console while running the simulation:

```shell
+UVM_VERBOSITY=UVM_HIGH
```

---

### Changing Verbosity for a Specific ID in a Class

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg::*;

// Driver class
class driver extends uvm_driver;
  `uvm_component_utils(driver)

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    `uvm_info("DRV1", "Executed Driver1 Code", UVM_HIGH);
    `uvm_info("DRV2", "Executed Driver2 Code", UVM_HIGH);
  endtask
endclass

module tb;
  driver drv;
  
  initial begin
    drv = new("DRV", null);
    drv.set_report_id_verbosity("DRV1", UVM_HIGH);
    drv.run();
  end
endmodule
```

---

### Changing Verbosity for an Entire Class Instead of Specific IDs

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg::*;

class driver extends uvm_driver;
  `uvm_component_utils(driver)

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    `uvm_info("DRV1", "Executed Driver1 Code", UVM_HIGH);
    `uvm_info("DRV2", "Executed Driver2 Code", UVM_HIGH);
  endtask
endclass

class env extends uvm_env;
  `uvm_component_utils(env)

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    `uvm_info("ENV1", "Executed ENV1 Code", UVM_HIGH);
    `uvm_info("ENV2", "Executed ENV2 Code", UVM_HIGH);
  endtask
endclass

module tb;
  driver drv;
  env e;
  
  initial begin
    drv = new("DRV", null);
    e = new("ENV", null);
    e.set_report_verbosity_level(UVM_HIGH);
    drv.run();
    e.run();
  end
endmodule
```

---

### Changing Verbosity for the Entire Hierarchy

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg.*;

class driver extends uvm_driver;
  `uvm_component_utils(driver)

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    `uvm_info("DRV", "Executed Driver Code", UVM_HIGH);
  endtask
endclass

class monitor extends uvm_monitor;
  `uvm_component_utils(monitor)

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    `uvm_info("MON", "Executed Monitor Code", UVM_HIGH);
  endtask
endclass

class env extends uvm_env;
  `uvm_component_utils(env)

  driver drv;
  monitor mon;

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    drv = new("DRV", this);
    mon = new("MON", this);
    drv.run();
    mon.run();
  endtask
endclass

module tb;
  env e;

  initial begin
    e = new("ENV", null);
    e.set_report_verbosity_level_hier(UVM_HIGH);
    e.run();
  end
endmodule
```

**Output:**

```
KERNEL: UVM_INFO /home/testbench.sv(14) @ 0: ENV.DRV [DRV] Executed the driver
KERNEL: UVM_INFO /home/testbench.sv(21) @ 0: ENV.MON [MON] Executed the monitor
```

---

### UVM Error and Warning Handling

UVM errors and warnings allow the simulation to continue, but a UVM_FATAL will stop the simulation.

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg.*;

class driver extends uvm_driver;
  `uvm_component_utils(driver)

  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction

  task run();
    `uvm_info("DRV", "Informational Message", UVM_NONE);
    `uvm_warning("DRV", "Potential Error");
    `uvm_error("DRV", "Real Error");
    #10;
    `uvm_fatal("DRV", "Simulation cannot continue");
  endtask
endclass

module tb;
  driver d;

  initial begin
    d = new("DRV", null);
    d.run();
  end
endmodule
```

**Simulation stops at 10ns due to UVM_FATAL. The reporting mechanisms display in different colors.**

---

### Setting a Limit to UVM Errors

You can set a limit on UVM errors, after which the simulation will automatically quit even without encountering a UVM_FATAL.

```systemverilog
module tb;
  driver d;

  initial begin
    d = new("DRV", null);
    d.set_report_max_quit_count(3);
    d.run();
  end
endmodule
```

---

### Creating a Log File

```systemverilog
module tb;
  driver d;
  int file;

  initial begin
    file = $fopen("log.txt", "w");
    d = new("DRV", null);
    
    // Store all messages in the terminal to a single file
    // d.set_report_default_file(file);
    
    // Store only messages of a particular type
    d.set_report_severity_file(UVM_ERROR, file);
    d.set_report_severity_action(UVM_ERROR, UVM_DISPLAY | UVM_LOG);
    
    d.run();
    
    #10;
    $fclose(file);
  end
endmodule
```
