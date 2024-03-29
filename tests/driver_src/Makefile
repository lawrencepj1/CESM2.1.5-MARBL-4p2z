.SUFFIXES: .F90 .o
extension = .F90

# DEFINE VARIOUS DIRECTORIES NEEDED FOR BUILD

# (1) The default directories are all defined relative to the location of this
#     Makefile

# Not all versions of gmake use MAKEFILE_LIST; if it exists then we determine
# the directory that contains the Makefile otherwise we assume Makefile is in
# the current working directory

# $(word n,VAR) looks for the nth word in $(VAR)
# so ThisMakefile is the first word in $(MAKEFILE_LIST) (which might be empty)
#override ThisMakefile=$(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
override ThisMakefile=$(word 1,$(MAKEFILE_LIST))

# if there are any / in $(ThisMakefile), strip everything after the last /
ifeq (/,$(findstring /,$(ThisMakefile)))
  override MAKE_DIR=$(realpath $(shell ThisMakefile=$(ThisMakefile) && echo $${ThisMakefile%/*}))
else
  override MAKE_DIR=$(realpath .)
endif

# (2) Default directories (can be changed by invoking "make LIB_SRC_DIR=...", etc)
OBJ_ROOT = $(realpath $(MAKE_DIR)/../obj)
INC_DIR = $(realpath $(MAKE_DIR)/../../include)
LIB_DIR = $(realpath $(MAKE_DIR)/../../lib)
EXE_DIR = $(realpath $(MAKE_DIR)/../driver_exe)
LIB_SRC_DIR = $(realpath $(MAKE_DIR)/../../src)
EXE_SRC_DIR = $(realpath $(MAKE_DIR)/.)

###################################################
# ERROR CHECKING: MAKE SURE ALL DIRECTORIES EXIST #
###################################################

ifeq ($(realpath $(OBJ_ROOT)),)
  $(error Can not find $$OBJ_ROOT)
endif

ifeq ($(realpath $(INC_DIR)),)
  $(error Can not find $$INC_DIR)
endif

ifeq ($(realpath $(LIB_DIR)),)
  $(error Can not find $$LIB_DIR)
endif

ifeq ($(realpath $(EXE_DIR)),)
  $(error Can not find $$EXE_DIR)
endif

ifeq ($(realpath $(LIB_SRC_DIR)),)
  $(error Can not find $$LIB_SRC_DIR)
endif

ifeq ($(realpath $(EXE_SRC_DIR)),)
  $(error Can not find $$EXE_SRC_DIR)
endif

##############################################
# DEFAULT VALUES FOR INTERNAL MAKE VARIABLES #
##############################################

COMP_NAME=NONE
USEMPI=FALSE
USE_NETCDF=FALSE

ifeq ($(COMP_NAME),NONE)
  LIB_NAME = libmarbl.a
  LFLAG = -lmarbl
else
  INC_LOC = $(realpath $(INC_DIR)/$(COMP_NAME))
  ifeq ($(realpath $(INC_LOC)),)
    $(error Can not find $$INC_LOC)
  endif

  OBJ_LOC = $(realpath $(OBJ_ROOT)/$(COMP_NAME))
  ifeq ($(realpath $(OBJ_LOC)),)
    $(error Can not find $$OBJ_LOC)
  endif

  OBJ_DIR = $(realpath $(OBJ_LOC)/driver)
  ifeq ($(realpath $(OBJ_DIR)),)
    $(error Can not find $$OBJ_DIR)
  endif

  LIB_NAME = libmarbl-$(COMP_NAME).a
  LFLAG = -lmarbl-$(COMP_NAME)
endif

# Determine if building with netCDF
ifneq ($(COMP_NAME),NONE)
  INFO_FILE = $(EXE_SRC_DIR)/.netcdf_info.$(COMP_NAME)
  ifeq ($(USE_NETCDF),TRUE)
    NETCDF_INC=-D_NETCDF -I$(shell nc-config --includedir)
    # hobart_intel version of nc-config does not have --libdir option
    NETCDF_LNK=-L$(shell nc-config --prefix)/lib -lnetcdf -lnetcdff
  endif

  # Determine if existing build used netCDF
  ifneq ($(wildcard $(INFO_FILE)),)
    OLD_NETCDF = $(shell cat $(INFO_FILE))
  else
    OLD_NETCDF = FORCE_REBUILD
  endif
endif

ifneq ($(OLD_NETCDF), $(USE_NETCDF))
  REBUILD = TRUE
endif

LIBCPPDEFS=
EXECPPDEFS=
INCLUDES=-I$(INC_LOC)
LINKS=-L$(LIB_DIR) $(LFLAG)

EXESRL = $(EXE_DIR)/marbl.exe
EXEMPI = $(EXE_DIR)/marbl-mpi.exe
ifeq ($(USEMPI),TRUE)
  EXECPPDEFS+= -DMARBL_TIMING_OPT=MPI
  LIBCPPDEFS+= -DMARBL_TIMING_OPT=MPI
  MPISUF=-mpi
  override FC = mpif90
  EXE = $(EXEMPI)
else
  EXE = $(EXESRL)
endif

COMP_ARGS = OBJ_DIR=$(OBJ_LOC) LIB_DIR=$(LIB_DIR) INC_DIR=$(INC_LOC) CPPDEFS="$(LIBCPPDEFS)"

OBJS = marbl_mpi_mod.o                      \
       marbl_tools_mod.o                    \
       marbl_netcdf_mod.o                   \
       marbl_io_mod.o                       \
       marbl_init_drv.o                     \
       marbl_call_compute_subroutines_drv.o \
       marbl_get_put_drv.o                  \
       marbl_utils_drv.o                    \
       marbl.o

MODS = marbl_mpi_mod.mod

###########
# TARGETS #
###########

# default is to build with gnu
.PHONY: all
all: gnu

.PHONY: gnu
gnu:
	$(MAKE) -f $(MAKE_DIR)/Makefile $(EXE) COMP_NAME=gnu$(MPISUF) FC=gfortran FCFLAGS="-Wall -Wextra -Wno-compare-reals -Werror -O2 -ffree-form -cpp -finit-real=snan -ffpe-trap=invalid" INC="-J $(OBJ_ROOT)/gnu$(MPISUF)" INC2="-J $(OBJ_ROOT)/gnu$(MPISUF)/driver"

.PHONY: intel
intel:
	$(MAKE) -f $(MAKE_DIR)/Makefile $(EXE) COMP_NAME=intel$(MPISUF) FC=ifort FCFLAGS="-g -O2 -init=snan,arrays -free -cpp -nogen-interface -fp-model source" INC="-module $(OBJ_ROOT)/intel$(MPISUF)" INC2="-module $(OBJ_ROOT)/intel$(MPISUF)/driver"

.PHONY:nvhpc
nvhpc:
	$(MAKE) -f $(MAKE_DIR)/Makefile $(EXE) COMP_NAME=nvhpc$(MPISUF) FC=nvfortran FCFLAGS="-O2 -Mfree" INC="-module $(OBJ_ROOT)/nvhpc$(MPISUF)" INC2="-module $(OBJ_ROOT)/nvhpc$(MPISUF)/driver"

.PHONY: pgi
pgi:
	$(MAKE) -f $(MAKE_DIR)/Makefile $(EXE) COMP_NAME=pgi$(MPISUF) FC=pgf90 FCFLAGS="-O2 -Mfree" INC="-module $(OBJ_ROOT)/pgi$(MPISUF)" INC2="-module $(OBJ_ROOT)/pgi$(MPISUF)/driver"

.PHONY: nag
nag:
	$(MAKE) -f $(MAKE_DIR)/Makefile $(EXE) COMP_NAME=nag$(MPISUF) FC=nagfor FCFLAGS="-colour -O2 -free -wmismatch=mpi_bcast" INC="-mdir $(OBJ_ROOT)/nag$(MPISUF) -I$(OBJ_ROOT)/nag$(MPISUF)" INC2="-mdir $(OBJ_ROOT)/nag$(MPISUF)/driver -I$(OBJ_ROOT)/nag$(MPISUF)/driver"

.PHONY: cray
cray:
	$(MAKE) -f $(MAKE_DIR)/Makefile $(EXE) COMP_NAME=cray FC=ftn FCFLAGS="-O2 -f free -e mf" INC="-J $(OBJ_ROOT)/cray" INC2="-J $(OBJ_ROOT)/cray/driver"

# Build the library
$(LIB_DIR)/%.a: $(LIB_SRC_DIR)/*.F90
	make -f $(LIB_SRC_DIR)/Makefile FC=$(FC) FCFLAGS="$(FCFLAGS) $(INC)" USE_DEPS=TRUE $(COMP_ARGS) $@

# Rebuild marbl_io_mod if marbl_netcdf_mod changes
$(OBJ_DIR)/marbl_io_mod.o: $(OBJ_DIR)/marbl_netcdf_mod.o
$(OBJ_DIR)/marbl_io_mod.mod: $(OBJ_DIR)/marbl_netcdf_mod.mod

# Rebuild marbl_call_compute_subroutines_drv if marbl_io_mod changes
$(OBJ_DIR)/marbl_call_compute_subroutines_drv.o: $(OBJ_DIR)/marbl_io_mod.o
$(OBJ_DIR)/marbl_call_compute_subroutines_drv.mod: $(OBJ_DIR)/marbl_io_mod.mod

# Driver object files
$(OBJ_DIR)/%.o: $(EXE_SRC_DIR)/%.F90 $(LIB_DIR)/$(LIB_NAME)
	$(FC) $(EXECPPDEFS) $(FCFLAGS) $(NETCDF_INC) $(INC2) $(INCLUDES) -c $< -o $@


# Determine if output needs to be rebuilt with or without netcdf
check:
	$(if $(REBUILD), rm -f $(OBJ_DIR)/marbl_netcdf_mod.*)

# Executable
$(EXE): check clean_exe                        \
        $(LIB_DIR)/$(LIB_NAME)           \
        $(addprefix $(OBJ_DIR)/,$(OBJS))
	$(FC) -o $(EXE) $(OBJ_DIR)/*.o $(LINKS) $(NETCDF_LNK)
	$(shell echo $(USE_NETCDF) > $(INFO_FILE))

# Clean up previous builds of the executable
.PHONY: clean_exe
clean_exe:
	rm -f $(EXESRL) $(EXEMPI)
	find $(OBJ_ROOT)/*/driver -name marbl.o | xargs rm -f

# Clean up previous builds of the driver
.PHONY: clean_driver
clean_driver: clean_exe
	find $(OBJ_ROOT)/*/driver -name marbl_*.o | xargs rm -f
	find $(OBJ_ROOT)/*/driver -name marbl_*.mod | xargs rm -f

.PHONY: clean
clean: clean_driver
	rm -f $(EXE_SRC_DIR)/.netcdf_info*
	make -f $(LIB_SRC_DIR)/Makefile clean
