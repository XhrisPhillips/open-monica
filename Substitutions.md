# Introduction #

Several classes such as email alerts and alarm guidance text use a standard method MonitorUtils.doSubstitutions which can replace special values in a template string with runtime variables such as the values of monitor points. This page serves to document the valid macro strings.


# Details #

The method signature looks like this:
```
public static String doSubstitutions(String template, PointData data, PointDescription point)
```

The arguments are as follows:
  * **template** the string which will have the macros substituted.
  * **data** The data which can be used for some of the substitutions. This can be null if required.
  * **point** The base point to which the substitutions should be applied.

The valid substitution strings in the template are:

|$V | The current value of the data. |
|:--|:-------------------------------|
|$V[point.name] | The current value of the specified point. |
|$U | The units of the data. |
|$N | The name of the point. |
|$S or $1 | The source part of the point name. |
|$D | The point's description. |
|$T | The data's timestamp as a UTC time. |
|$A | The alarm status of the data (true or false). |
|$a | The alarm status of the data (ALARMING or OK). |