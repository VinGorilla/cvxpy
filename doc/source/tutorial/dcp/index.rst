.. _dcp:

Expressions
-----------

Expressions in CVXPY are formed from variables, parameters, numerical
constants such as Python floats and Numpy matrices, the standard
arithmetic operators ``+, -, *, /``, and a library of
`functions </functions>`__. Here are some examples of CVXPY expressions:

.. code:: python

    from cvxpy import *

    # Create variables and parameters.
    x, y = Variable(), Variable()
    a, b = Parameter(), Parameter()

    # Examples of CVXPY expressions.
    3.69 + b/3
    x - 4*a
    sqrt(x) - min(y, x - a)
    max(2.66 - sqrt(y), square(x + 2*y))



.. parsed-literal::

    max(2.66 + -sqrt(var15), square(var14 + 2 * var15))



Expressions can be scalars, vectors, or matrices. Use ``expr.size`` to
see the dimensions of an expression. You can also use the helper methods
``expr.is_scalar()`` and ``expr.is_vector()`` to test if the expression
is a scalar or vector (1xN or Nx1). CVXPY will raise an exception if an
expression is used in a way that doesn't make sense given its
dimensions, for example adding matrices of different size.

.. code:: python

    import numpy

    X = Variable(5, 4)
    A = numpy.ones((3, 5))

    # Use expr.size to get the dimensions.
    print "dimensions of X:", X.size
    print "dimensions of sum(X):", sum(X).size
    print "dimensions of A*X:", (A*X).size

    # Helper methods.
    print "sum(X) is a scalar:", sum(X).is_scalar()
    print "A*X is a vector:", (A*X).is_vector()

    # ValueError raised for invalid dimensions.
    try:
        A + X
    except ValueError, e:
        print e

.. parsed-literal::

    dimensions of X: (5, 4)
    dimensions of sum(X): (1, 1)
    dimensions of A*X: (3, 4)
    sum(X) is a scalar: True
    A*X is a vector: False
    Incompatible dimensions (3, 5) (5, 4)


Disciplined Convex Programming
------------------------------

CVXPY uses Disciplined Convex Programming (DCP) to determine the sign
and curvature of each expression. The following sections explain how DCP
works and how to get sign and curvature information from a CVXPY
expression. Visit `dcp.stanford.edu <http://dcp.stanford.edu>`__ for a
more interactive introduction to DCP.

Sign
----

Each (sub)expression is flagged as *positive* (non-negative), *negative*
(non-positive), *zero*, or *unknown*.

The signs of larger expressions are determined from the signs of their
subexpressions. For example, the sign of the expression expr1\*expr2 is

-  Zero if either expression has sign zero.
-  Positive if expr1 and expr2 have the same (known) sign.
-  Negative if expr1 and expr2 have opposite (known) signs.
-  Unknown if either expression has unknown sign.

The sign given to an expression is always correct. But DCP sign analysis
may flag an expression as unknown sign when the sign could be figured
out through more complex analysis. For instance, ``x*x`` is positive but
has unknown sign by the rules above.

The sign of an expression is stored as ``expr.sign``. A vector or matrix
expression may have entries with different signs, in which case
``expr.sign`` is a Numpy 2D array containing the signs of all the
entries.

.. code:: python

    x = Variable()
    a = Parameter(sign="negative")
    c = numpy.array([1, -1])

    print "sign of x:", x.sign
    print "sign of a:", a.sign
    print "sign of square(x)", square(x).sign
    print "sign of c*a"
    print (c*a).sign

.. parsed-literal::

    sign of x: UNKNOWN
    sign of a: NEGATIVE
    sign of square(x) POSITIVE
    sign of c*a
    [['NEGATIVE']
     ['POSITIVE']]


Curvature
---------

Each (sub)expression is flagged as one of the following curvatures

.. raw:: html

   <table>
   <tr>
    <th>

Curvature

.. raw:: html

   </th>
    <th>

Meaning

.. raw:: html

   </th>
   </tr>
   <tr>
     <td>

constant

.. raw:: html

   </td>
     <td>

$ f(x) $ independent of $ x $

.. raw:: html

   </td>
   </tr>
   <tr>
     <td>

affine

.. raw:: html

   </td>
     <td>

$ f(x + (1-)y) = f(x) + (1-)f(y) $

.. raw:: html

   </td>
   </tr>
   <tr>
     <td>

convex

.. raw:: html

   </td>
     <td>

$ f(x + (1-)y) f(x) + (1-)f(y) $

.. raw:: html

   </td>
   </tr>
   <tr>
     <td>

concave

.. raw:: html

   </td>
     <td>

$ f(x + (1-)y) f(x) + (1-)f(y) $

.. raw:: html

   </td>
   </tr>
   <tr>
     <td>

unknown

.. raw:: html

   </td>
     <td>

DCP analysis cannot determine the curvature

.. raw:: html

   </td>
   </tr>
   </table>

using the curvature rules given below. As with sign analysis, the
conclusion is always correct, but the simple analysis can flag
expressions as unknown even when they are convex or concave. Note that
any constant expression is also affine, and any affine expression is
convex and concave.

Curvature Rules
---------------

DCP analysis is based on applying a general composition theorem from
convex analysis to each (sub)expression.

:math:`f(expr_1, expr_2, ..., expr_n)` is convex if :math:`\text{ } f`
is a convex function and for each :math:`expr_{i}` one of the following
conditions holds:

-  :math:`f` is increasing in argument i and :math:`expr_{i}` is convex.
-  :math:`f` is decreasing in argument i and :math:`expr_{i}` is
   concave.
-  :math:`expr_{i}` is affine or constant.

:math:`f(expr_1, expr_2, ..., expr_n)` is concave if :math:`\text{ } f`
is a concave function and for each :math:`expr_{i}` one of the following
conditions holds:

-  :math:`f` is increasing in argument i and :math:`expr_{i}` is
   concave.
-  :math:`f` is decreasing in argument i and :math:`expr_{i}` is convex.
-  :math:`expr_{i}` is affine or constant.

:math:`f(expr_1, expr_2, ..., expr_n)` is affine if :math:`\text{ } f`
is an affine function function and each :math:`expr_{i}` is affine.

If none of the three rules apply, the expression
:math:`f(expr_1, expr_2, ..., expr_n)` is marked as having unknown
curvature.

Whether a function is increasing or decreasing in an argument may depend
on the sign of the argument. For instance, ``square`` is increasing for
positive arguments and decreasing for negative arguments.

The curvature of an expression is stored as ``expr.curvature``. A vector
or matrix expression may have entries with different curvatures, in
which case ``expr.curvature`` is a Numpy 2D array containing the
curvatures of all the entries.

.. code:: python

    x = Variable()
    a = Parameter(sign="positive")
    c = numpy.array([1, -1])

    print "curvature of x:", x.curvature
    print "curvature of a:", a.curvature
    print "curvature of square(x)", square(x).curvature
    print "curvature of c*square(x)"
    print (c*square(x)).curvature

.. parsed-literal::

    curvature of x: AFFINE
    curvature of a: CONSTANT
    curvature of square(x) CONVEX
    curvature of c*square(x)
    [['CONVEX']
     ['CONCAVE']]


Infix Operators
---------------

The infix operators ``+, -, *, /`` are treated exactly like functions.
The infix operators ``+`` and ``-`` are affine, so the rules above are
used to flag the curvature. For example, ``expr1 + expr2`` is flagged as
convex if ``expr1`` and ``expr2`` are convex.

``expr1*expr2`` is allowed only when one of the expressions is constant.
If both expressions are non-constant, CVXPY will raise an exception.
``expr1/expr2`` is allowed only when ``expr2`` is a scalar constant. The
curvature rules above apply. For example, ``expr1/expr2`` is convex when
``expr1`` is concave and ``expr2`` is negative and constant.

Example 1
---------

DCP analysis breaks expressions down into subexpressions. The tree
visualization below shows how this works for the expression
``2*square(x) + 3``. Each subexpression is shown in a blue box. We mark
its curvature on the left and its sign on the right.

Example 2
---------

We'll walk through the application of the DCP rules to the expression
``sqrt(1 + square(x))``.

The variable ``x`` has affine curvature and unknown sign. The ``square``
function is convex and non-monotone for arguments of unknown sign. It
can take the affine expression ``x`` as an argument; the result
``square(x)`` is convex.

The arithmetic operator ``+`` is affine and increasing, so the
composition ``1 + square(x)`` is convex by the curvature rule for convex
functions. The function ``sqrt`` is concave and increasing, which means
it can only take a concave argument. Since ``1 + square(x)`` is convex,
``sqrt(1 + square(x))`` violates the DCP rules and cannot be verified as
convex.

In fact, ``sqrt(1 + square(x))`` is a convex function of ``x``, but the
DCP rules are not able to verify convexity. If the expression is written
as ``norm(vstack(1, x), 2)``, the L2 norm of the vector :math:`[1,x]`,
which has the same value as ``sqrt(1 + square(x))``, then it will be
certified as convex using the DCP rules.

.. code:: python

    print "curvature of sqrt(1 + square(x))", sqrt(1 + square(x)).curvature
    print "curvature of norm(vstack(1, x), 2)", norm(vstack(1, x), 2).curvature

.. parsed-literal::

    curvature of sqrt(1 + square(x)) UNKNOWN
    curvature of norm(vstack(1, x), 2) CONVEX

