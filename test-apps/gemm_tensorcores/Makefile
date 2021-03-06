GPU=1
DEBUG=0
#for radiation setup
LOGS=1
OMP=1

ALPHA=1.0
BETA=0.0
#Fault injection uses size table
# half cublas is 16k to 1k
# single cublas is 16k to 512
# double cublas is 16k to 4k
SIZE?=1024
ITERATIONS=1
PRECISION=half
CUBLAS=1
TENSOR_CORES=0
DMR=none
BUILDRELATIVEERROR=0
CHECKBLOCK=1

RADDIR=/home/carol/radiation-benchmarks
OUTPUTPREFIX=$(PRECISION)_$(ALPHA)_$(BETA)_$(SIZE)_cublas_$(CUBLAS)_tensor_$(TENSOR_CORES).matrix


# This is what I use, uncomment if you know your arch and want to specify
ARCH= 	-gencode arch=compute_35,code=[sm_35,compute_35]		# Kepler
#ARCH+= 	-gencode arch=compute_61,code=[sm_61,compute_61]	# Titan X | Titan Xp
#ARCH+= 	-gencode arch=compute_62,code=[sm_62,compute_62]	# Tegra X2
ARCH+= 	-gencode arch=compute_70,code=[sm_70,compute_70]	# Titan V | Tesla V100
#ARCH+= 	-gencode arch=compute_72,code=[sm_72,compute_72]	# Xavier


VPATH=./src/
TARGET=gemm
OBJDIR=./obj/

CXX=g++
#g++
CUDAPATH=/usr/local/cuda
NVCC=$(CUDAPATH)/bin/nvcc 
OPTS=-O3
LDFLAGS= -lm -pthread -lstdc++ 
COMMON= 
CFLAGS=-Wall -Wfatal-errors  -Wunknown-pragmas -Wunused-function
RAD_DIR=/home/carol/radiation-benchmarks

INCLUDE=-I$(CUDAPATH)/include -I$(RADDIR)/src/cuda/common 
# -Icutlass/include

ifeq ($(CUBLAS), 1)
USECUBLAS= --use_cublas
endif

ifeq ($(TENSOR_CORES), 1)
USETENSORS= --tensor_cores
endif

ifeq ($(DEBUG), 1) 
OPTS=-O0 -g -DDEBUG
NVCCFLAGS+= -g -G -DDEBUG
endif

ifeq ($(BUILDRELATIVEERROR), 1)
NVCCFLAGS+= -DBUILDRELATIVEERROR=1
endif

CFLAGS+=$(OPTS)
STDVERSION=--std=c++11

COMMON+= $(STDVERSION)

ifeq ($(GPU), 1) 
COMMON+= -DGPU -I$(CUDAPATH)/include/
CFLAGS+= -DGPU
LDFLAGS+= -L$(CUDAPATH)/lib64 -lcudart -lcurand -lcublas
endif

ifeq ($(OMP), 1) 
CFLAGS+= -DOMP -fopenmp
endif


OBJ= main.o setup_gemm.o Parameters.o common.o
#setup_tensor_cores.o

ifeq ($(LOGS), 1)
INCLUDE+=-I$(RADDIR)/src/include/
NVCCFLAGS+= -DLOGS
CFLAGS+= -DLOGS
LDFLAGS+= -L$(RADDIR)/src/include/ -lLogHelper -DLOGS=1
endif

OBJS = $(addprefix $(OBJDIR), $(OBJ))
DEPS = $(wildcard src/*.h) Makefile

NVCCFLAGS+= -Xptxas -v

all: obj $(TARGET)

$(TARGET): $(OBJS)
	$(CXX) $(COMMON) $(CFLAGS) $^ $(INCLUDE) -o $@  $(LDFLAGS)  

$(OBJDIR)%.o: %.cpp $(DEPS)
	$(CXX) $(COMMON) $(CFLAGS) -c $< -o $@ $(INCLUDE) 

$(OBJDIR)%.o: %.cu $(DEPS)
	$(NVCC) -ccbin $(CXX) $(ARCH) $(COMMON) $(NVCCFLAGS) --compiler-options "$(CFLAGS)" $(INCLUDE) -c $< -o $@ 


obj:
	mkdir -p obj


.PHONY: clean
clean:
	rm -rf $(OBJS) $(TARGET)

# GEMM options
generate:
	./$(TARGET) --size $(SIZE) --generate --verbose \
			--precision $(PRECISION) --dmr $(DMR) \
			--alpha $(ALPHA) --beta $(BETA) \
			--input_a $(RADDIR)/data/gemm/a_$(OUTPUTPREFIX) \
			--input_b $(RADDIR)/data/gemm/b_$(OUTPUTPREFIX) \
			--input_c $(RADDIR)/data/gemm/c_$(OUTPUTPREFIX) \
			--gold    $(RADDIR)/data/gemm/g_$(OUTPUTPREFIX) $(USECUBLAS) $(USETENSORS)


test:
	./$(TARGET) --size $(SIZE) --verbose \
			--precision $(PRECISION) --dmr $(DMR) \
			--iterations $(ITERATIONS)  \
			--alpha $(ALPHA) --beta $(BETA) \
			--input_a $(RADDIR)/data/gemm/a_$(OUTPUTPREFIX) \
			--input_b $(RADDIR)/data/gemm/b_$(OUTPUTPREFIX) \
			--input_c $(RADDIR)/data/gemm/c_$(OUTPUTPREFIX) \
			--gold    $(RADDIR)/data/gemm/g_$(OUTPUTPREFIX) $(USECUBLAS) $(USETENSORS)

golden:
	./$(TARGET) --size $(SIZE) --verbose \
			--precision $(PRECISION) --dmr $(DMR) \
			--iterations $(ITERATIONS)  \
			--alpha $(ALPHA) --beta $(BETA) \
			--input_a $(RADDIR)/data/gemm/a_$(OUTPUTPREFIX) \
			--input_b $(RADDIR)/data/gemm/b_$(OUTPUTPREFIX) \
			--input_c $(RADDIR)/data/gemm/c_$(OUTPUTPREFIX) \
			--gold    $(RADDIR)/data/gemm/g_$(OUTPUTPREFIX) $(USECUBLAS) $(USETENSORS)  > golden_stdout.txt 2>golden_stderr.txt
	sed -i '/LOGFILENAME/c\' golden_stdout.txt 
	sed -i '/Time/c\' golden_stdout.txt 
	sed -i '/time/c\' golden_stdout.txt 
	sed -i '/^$$/d' golden_stdout.txt

