# Schemdraw Guide

## Installation

```bash
pip install schemdraw
# Optional dependencies for enhanced features
pip install matplotlib numpy
# For LaTeX support (optional)
pip install latex
```

## Overview

**Specialty:** Circuit diagrams and technical schematics

**Best Use Cases:**
- Electrical circuit diagrams
- Electronic schematic drawings
- Signal flow diagrams
- Control system block diagrams
- Technical documentation for engineering
- Educational circuit illustrations

## ASCII Example

Below is an ASCII representation of what we'll create - an electronic circuit diagram:

```
    +12V
     │
     ├─────[R1]─────┐
     │              │
    [C1]           [LED]
     │              │
     ├─────[R2]─────┤
     │              │
    GND            GND

Components:
- R1: 1kΩ resistor
- R2: 330Ω resistor  
- C1: 100μF capacitor
- LED: Light emitting diode
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
import schemdraw
import schemdraw.elements as elm
import schemdraw.dsp as dsp
import schemdraw.logic as logic
```

### 2. Prepare Your Data

```python
# Define circuit parameters
components = {
    'R1': {'value': '1kΩ', 'type': 'resistor'},
    'R2': {'value': '330Ω', 'type': 'resistor'},
    'C1': {'value': '100μF', 'type': 'capacitor'},
    'LED1': {'value': 'Red LED', 'type': 'led'},
    'VCC': {'value': '+12V', 'type': 'voltage_source'}
}

# Circuit specifications
circuit_title = "LED Driver Circuit"
```

### 3. Create the Basic Visualization

```python
# Create a basic LED driver circuit
with schemdraw.Drawing() as d:
    d.config(fontsize=12)
    
    # Voltage source
    d += elm.SourceV().up().label('+12V')
    d += elm.Line().right()
    
    # Branch for LED path
    d.push()  # Save current position
    d += elm.Resistor().down().label('R1\n1kΩ')
    d += elm.LED().down().label('LED')
    d += elm.Line().left()
    
    # Return to saved position for capacitor branch
    d.pop()
    d += elm.Line().right()
    d += elm.Capacitor().down().label('C1\n100μF')
    d += elm.Resistor().down().label('R2\n330Ω')
    d += elm.Line().left()
    
    # Ground connection
    d += elm.Ground()
    
    # Add title
    d.draw()
    d.save('basic_led_circuit.png')
```

### 4. Customize the Appearance

```python
# Create an enhanced circuit with better styling
with schemdraw.Drawing(show=False) as d:
    d.config(fontsize=14, color='navy')
    
    # Set drawing parameters
    d += elm.Gap().label('LED Driver Circuit', loc='top', fontsize=16, color='black')
    
    # Power supply section
    d += elm.SourceV().up().label('+12V').color('red')
    d += elm.Line().right().color('red')
    
    # Current limiting resistor
    d.push()
    d += elm.Resistor().down().label('R1\n1kΩ\n(Current Limit)', loc='left').color('blue')
    
    # LED with proper symbol
    d += elm.LED().down().label('D1\nRed LED\n2V @ 20mA', loc='right').color('red')
    
    # Return path
    d += elm.Line().left().color('black')
    
    # Parallel capacitor branch
    d.pop()
    d += elm.Line().right(2).color('red')
    d += elm.Capacitor().down().label('C1\n100μF\nFiltering', loc='right').color('green')
    d += elm.Resistor().down().label('R2\n330Ω\nBleeder', loc='right').color('blue')
    d += elm.Line().left(2).color('black')
    
    # Ground symbol
    d += elm.Ground().color('black')
    
    # Add voltage and current annotations
    d += elm.CurrentLabel(direction='in').at(d.elements[-6].start).label('I = 10mA')
    
    d.draw()
    d.save('enhanced_led_circuit.png')
```

### 5. Create Complex Technical Diagrams

```python
# Create a more complex amplifier circuit
with schemdraw.Drawing(show=False) as d:
    d.config(fontsize=12)
    
    # Input section
    d += elm.Gap().label('Audio Amplifier Circuit', loc='top', fontsize=16)
    d += elm.Dot().label('Input', loc='left')
    d += elm.Capacitor().right().label('C1\n1μF')
    
    # Amplifier stage
    d += elm.Opamp(leads=True).anchor('in1').label('LM358')
    
    # Feedback network
    d.push()
    d += elm.Line().up().at(d.elements[-1].out)
    d += elm.Resistor().left().label('Rf\n10kΩ')
    d += elm.Line().down().to(d.elements[-2].in2)
    
    # Input bias
    d.pop()
    d += elm.Resistor().down().at(d.elements[-1].in2).label('R1\n1kΩ')
    d += elm.Ground()
    
    # Output section
    d += elm.Line().right().at(d.elements[-4].out)
    d += elm.Capacitor().right().label('C2\n10μF')
    d += elm.Dot().label('Output', loc='right')
    
    # Power supply connections
    d += elm.Line().up().at(d.elements[-6].vs)
    d += elm.SourceV().up().label('+9V')
    d += elm.Ground()
    
    d.draw()
    d.save('amplifier_circuit.png')

# Create a digital logic diagram
with schemdraw.Drawing(show=False) as d:
    d.config(fontsize=12)
    
    # Logic gates
    d += elm.Gap().label('Digital Logic Circuit', loc='top', fontsize=16)
    
    # Inputs
    d += elm.Dot().label('A', loc='left')
    d.push()
    d += elm.Line().right()
    d += logic.And().anchor('in1').label('AND1')
    
    d.pop()
    d += elm.Line().down()
    d += elm.Dot().label('B', loc='left')
    d += elm.Line().right().to(d.elements[-2].in2)
    
    # Second gate
    d += elm.Line().right().at(d.elements[-2].out)
    d += logic.Or().anchor('in1').label('OR1')
    
    # Third input
    d += elm.Dot().at(d.elements[-1].in2).label('C', loc='left')
    d += elm.Line().left()
    
    # Output
    d += elm.Line().right().at(d.elements[-3].out)
    d += elm.Dot().label('Output', loc='right')
    
    d.draw()
    d.save('logic_circuit.png')
```

### 6. Export or Display

```python
# Create a comprehensive technical diagram with multiple formats
with schemdraw.Drawing(show=False) as d:
    d.config(fontsize=14, inches_per_unit=0.5)
    
    # Create a complete system diagram
    d += elm.Gap().label('Complete System Schematic', loc='top', fontsize=18)
    
    # Power supply section
    d += elm.SourceV().up().label('VCC\n+5V')
    d += elm.Line().right(2)
    
    # Microcontroller representation
    d.push()
    d += elm.Ic(pins=[
        elm.IcPin('VCC', side='top'),
        elm.IcPin('GND', side='bottom'),
        elm.IcPin('GPIO1', side='right'),
        elm.IcPin('GPIO2', side='right')
    ]).label('MCU')
    
    # LED outputs
    d += elm.Line().right().at(d.elements[-1].GPIO1)
    d += elm.Resistor().right().label('220Ω')
    d += elm.LED().right().label('Status LED')
    d += elm.Ground()
    
    d += elm.Line().right().at(d.elements[-5].GPIO2)
    d += elm.Line().down()
    d += elm.Resistor().right().label('1kΩ')
    d += elm.LED().right().label('Power LED')
    d += elm.Ground()
    
    # Power connections
    d += elm.Line().up().at(d.elements[-9].VCC)
    d.pop()
    d += elm.Line().down().at(d.elements[-1].GND)
    d += elm.Ground()
    
    # Save in multiple formats
    d.save('complete_system.png')
    d.save('complete_system.svg')
    d.save('complete_system.pdf')
    
    d.draw()

print("Circuit diagrams created successfully!")
print("Files generated:")
print("- basic_led_circuit.png")
print("- enhanced_led_circuit.png") 
print("- amplifier_circuit.png")
print("- logic_circuit.png")
print("- complete_system.png/svg/pdf")
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Overlapping components in complex circuits
  - Solution: Use `d.push()` and `d.pop()` to manage drawing position, plan layout carefully
- **Pitfall 2**: Inconsistent component sizing
  - Solution: Use `d.config()` to set global parameters and consistent spacing
- **Pitfall 3**: Poor label placement
  - Solution: Use `loc` parameter ('left', 'right', 'top', 'bottom') for optimal positioning

## Advanced Techniques

- Creating custom component symbols for specialized applications
- Using mathematical expressions in labels with LaTeX support
- Building reusable circuit blocks as functions
- Implementing automatic wire routing for complex layouts
- Creating animated circuit diagrams showing signal flow
- Integrating with circuit simulation tools for verification

## Resources

- [Schemdraw Documentation](https://schemdraw.readthedocs.io/en/latest/)
- [Schemdraw Gallery](https://schemdraw.readthedocs.io/en/latest/gallery/index.html)
- [Circuit Drawing Best Practices](https://schemdraw.readthedocs.io/en/latest/usage/start.html)
- [Electronic Schematic Standards](https://en.wikipedia.org/wiki/Electronic_symbol)
