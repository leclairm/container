Image structure
===============

General Patterns
----------------

Install dummy slurm
^^^^^^^^^^^^^^^^^^^
COSMO and INT2LM both have slurm as a runtime dependency. Since we only use the binaries, but not any infrastructure from within the container,
slurm is not used. To save build-time a dummy slurm installation is passed to spack via ``packages.yaml``

.. code-block:: Docker
                
   RUN echo "  slurm:" >> /root/.spack/packages.yaml && \
       echo "      buildable: false" >> /root/.spack/packages.yaml && \
       echo "      externals:" >> /root/.spack/packages.yaml && \
       echo "      - spec: slurm%gcc" >> /root/.spack/packages.yaml && \
       echo "        prefix: /usr" >> /root/.spack/packages.yaml

Load runtime environment with Spack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to load the correct environment variables at runtime, i.e. ``GRIB_DEFINITION_PATH``
the command ``spack load --sh`` is written to ``/etc/profile``:

.. code-block:: Docker
                
   # dump spack-env to file
   RUN echo $(spack load --sh $COSMO_SPEC) > /opt/spack-env

The content of ``/opt/spack-env`` is copied to the lightweight runtime image and added to ``/etc/profile``:

.. code-block:: Docker
                
   COPY --from=builder /opt/spack-env /opt/spack-env

   ...

   # put spack-env into profile
   RUN echo "$(cat opt/spack-env)" >> /etc/profile

Finally the runtime enviromnment is loaded in the entrypoint of the Dockerfile:

.. code-block:: Docker
                
   ENTRYPOINT ["/bin/bash", "--rcfile", "/etc/profile", "-l" , "-c"]

Dockerfiles
-----------

.. digraph:: TD

   nvidia-spack -> mpich;
   mpich -> cosmo:cpu;
   mpich -> cosmo:gpu;
   mpich -> int2lm;
