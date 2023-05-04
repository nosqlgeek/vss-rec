# Redis as a Vector Database for Recommendations

This repository contains example code for using Redis Stack's vector similarity search as a recommendation engine.

The idea is quite simple:

1. The interests of a user are expressed as a vector
2. Each component of the vector is associated to one category of interest

The table following table shows the vector `[0.9, 0.7, 0.2]`:

|books|comics|computers|
|---|---|---|
|0.9|0.7|0.2|



Furthermore, I am stating that a user is only interested in a category if the component value is larger than the threshold of 0.4.

