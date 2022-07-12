Image structure
===============

General Patterns
----------------

Install dummy slurm
^^^^^^^^^^^^^^^^^^^
COSMO and INT2LM both have slurm as a runtime dependency. Since we onle use the binaries, but not any infrastructure from withing the container,
slurm is not used. To save build-time a dummy slurm installation is passed to spack via :code:`packages.yaml`

.. code-block:: Docker
                
   RUN echo "  slurm:" >> /root/.spack/packages.yaml && \
   echo "      buildable: false" >> /root/.spack/packages.yaml && \
   echo "      externals:" >> /root/.spack/packages.yaml && \
   echo "      - spec: slurm%gcc" >> /root/.spack/packages.yaml && \
   echo "        prefix: /usr" >> /root/.spack/packages.yaml

Load runtime environment with Spack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to load the correct environment variables at runtime, i.e. :code:`GRIB_DEFINITION_PATH`
the command :code:`spack load --sh` is written to :code:`/etc/profile`:

.. code-block:: Docker
                
   # dump spack-env to file
   RUN echo $(spack load --sh $COSMO_SPEC) > /opt/spack-env
