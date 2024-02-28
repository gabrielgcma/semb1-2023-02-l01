# Sistemas Embarcados I - 2023/3 - Universidade Federal de Uberlândia
# Aluno: Gabriel Carneiro Marques Amado
# Matrícula: 12111ECP002

CC = arm-none-eabi-gcc
CFLAGS = -g -mcpu=cortex-m4 -mthumb -O0 -Wall 

OBJDIR = build

DEPDIR = .deps
DEPFLAGS = -MMD -MP -MF $(DEPDIR)/$*.d

# Pega todos os .c do ./src/ 
SRCS = $(basename $(notdir $(wildcard ./src/*.c)))
# $(info $(SRCS))

OBJS = $(patsubst %, $(OBJDIR)/%.o, $(SRCS))
# $(info $(OBJS))
$(shell mkdir -p $(dir $(OBJS)) > /dev/null)

DEPS = $(patsubst %, $(DEPDIR)/%.d, $(SRCS))
# $(info $(DEPS))
$(shell mkdir -p $(dir $(DEPS)) > /dev/null)

all: $(OBJS) 

# $< = primeiro elemento da lista de pré requisitos
# $@ = alvo da regra

$(OBJDIR)/%.o: ./src/%.c $(DEPDIR)/%.d
	$(CC) -c $(CFLAGS) $(DEPFLAGS) $< -o $@

$(DEPS):
-include $(DEPS)

.PHONY: clean
clean:
	rm -rf $(OBJDIR) $(DEPDIR)
