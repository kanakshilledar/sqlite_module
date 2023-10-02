#
# Copyright (c) 2017 Intel Corporation
# Copyright (c) 2022 Nordic Semiconductor ASA
#
# SPDX-License-Identifier: Apache-2.0
#

PREFIX ?= .
OBJ_DIR ?= $(PREFIX)/obj
LIB_DIR ?= $(PREFIX)/lib

ifeq ($(findstring Windows, $(OS)),Windows)
	DEL := rmdir /S /Q
	MKDIR := mkdir
else
	DEL := rm -rf
	MKDIR := mkdir -p
endif

all:
	-$(MKDIR) "$(OBJ_DIR)"
	-$(MKDIR) "$(LIB_DIR)"
	$(CC) -c $(CFLAGS) -MD -Iinclude src/sqlite3.c -o $(OBJ_DIR)/sqlite3.o
	$(AR) -rcs $(LIB_DIR)/libsqlite3.a $(OBJ_DIR)/sqlite3.o

	$(CC) -c $(CFLAGS) -MD -Iinclude src/unishox1.c -o $(OBJ_DIR)/unishox1.o
	$(AR) -rcs $(LIB_DIR)/libunishox1.a $(OBJ_DIR)/unishox1.o

	$(CC) -c $(CFLAGS) -MD -Iinclude src/shox96_0_2.cpp -o $(OBJ_DIR)/shox96_0_2.o
	$(AR) -rcs $(LIB_DIR)/libshox96_0_2.a $(OBJ_DIR)/shox96_0_2.o

clean:
	$(DEL) "$(OBJ_DIR)" "$(LIB_DIR)"
