====================
QISKit Documentation
====================

Quantum Information Software Kit (QISKit), SDK Python version for working
with `OpenQASM <https://github.com/IBM/qiskit-openqasm>`_ and the IBM Q experience (QX).

Philosophy
==========

The basic concept of our quantum program is an array of quantum
circuits. The program workflow consists of three stages: Build, Compile,
and Run. Build allows you to make different quantum circuits that
represent the problem you are solving; Compile allows you to rewrite
them to run on different backends (simulators/real chips of different
quantum volumes, sizes, fidelity, etc); and Run launches the jobs. After
the jobs have been run, the data is collected. There are methods for
putting this data together, depending on the program. This either gives
you the answer you wanted or allows you to make a better program for the
next instance.

Project Overview
================
The QISKit project comprises:

* `QISKit SDK <https://github.com/IBM/qiskit-sdk-py>`_: Provides
  support for the Quantum Experience circuit generation phase and lets
  you use the QISKit API to access the Quantum Experience hardware and
  simulators. The SDK also includes example scripts written for
  Jupyter Notebooks.

* `QISKit API <https://github.com/IBM/qiskit-api-py>`_: A thin Python
  wrapper around the Quantum Experience HTTP API that enables you to
  connect and and execute OpenQASM code.

* `QISKit OpenQASM <https://github.com/IBM/qiskit-openqasm>`_: Contains
  specifications, examples, documentation, and tools for the OpenQASM
  intermediate representation.


Getting Started
===============

The starting point for writing code is the QuantumProgram object. The
QuantumProgram is a collection of circuits, or scores if you are
coming from the Quantum Experience, quantum register objects, and
classical register objects. The QuantumProgram methods can send these
circuits to quantum hardware or simulator backends and collect the
results for further analysis.

To compose and run a circuit on a simulator, which is distributed with
this project, one can do,

.. code-block:: python

   from qiskit import QuantumProgram
   qp = QuantumProgram()
   qr = qp.create_quantum_register('qr', 2)
   cr = qp.create_classical_register('cr', 2)
   qc = qp.create_circuit('Bell', [qr], [cr])
   qc.h(qr[0])
   qc.cx(qr[0], qr[1])
   qc.measure(qr[0], cr[0])
   qc.measure(qr[1], cr[1])
   result = qp.execute('Bell')
   print(result.get_counts('Bell'))

The :code:`get_counts` method outputs a dictionary of state:counts pairs;

.. code-block:: python

	 {'00': 531, '11': 493}

Project Organization
--------------------

Python example programs can be found in the *examples* directory, and test scripts are
located in *test*. The *qiskit* directory is the main module of the SDK.

Structure
=========

Programming interface
---------------------

The *qiskit* directory is the main Python module and contains the
programming interface objects `QuantumProgram <quantum_program.rst>`__, *QuantumRegister*,
*ClassicalRegister*, and *QuantumCircuit*.

At the highest level, users construct a *QuantumProgram* to create,
modify, compile, and execute a collection of quantum circuits. Each
*QuantumCircuit* has a set of data registers, each of type
*QuantumRegister* or *ClassicalRegister*. Methods of these objects are
used to apply instructions that define the circuit. The *QuantumCircuit*
can then generate **OpenQASM** code that can flow through other
components in the *qiskit* directory.

The *extensions* directory extends quantum circuits as needed to support
other gate sets and algorithms. Currently there is a *standard*
extension defining some typical quantum gates.

Internal modules
----------------

The directory also contains internal modules that are still under
development:

-  a *qasm* module for parsing **OpenQASM** circuits
-  an *unroll* module to interpret and "unroll" **OpenQASM** to a target
   gate basis (expanding gate subroutines and loops as needed)
-  a *circuit* module for working with circuits as graphs
-  a *mapper* module for mapping all-to-all circuits to run on devices
   with fixed couplings

Quantum circuits flow through the components as follows. The programming
interface is used to generate **OpenQASM** circuits. **OpenQASM**
source, as a file or string, is passed into a *Qasm* object, whose
*parse* method produces an abstract syntax tree (**AST**). The **AST**
is passed to an *Unroller* that is attached to an *UnrollerBackend*.
There is a *PrinterBackend* for outputting text, a *SimulatorBackend*
for outputting simulator input data for the local simulators, and a
*CircuitBackend* for constructing *Circuit* objects. The *Circuit*
object represents an "unrolled" **OpenQASM** circuit as a directed
acyclic graph (**DAG**). The *Circuit* provides methods for
representing, transforming, and computing properties of a circuit and
outputting the results again as **OpenQASM**. The whole flow is used by
the *mapper* module to rewrite a circuit to execute on a device with
fixed couplings given by a *CouplingGraph*.

The four circuit representations and how they are currently transformed
into each other are summarized in this figure:



.. image:: ../images/circuit_representations.png
    :width: 600px
    :align: center

Several unroller backends and their outputs are summarized here:



.. image:: ../images/unroller_backends.png
    :width: 600px
    :align: center

