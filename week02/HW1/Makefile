PROG=   bbcp
OBJS=   bbcp.o

all: ${PROG}

${PROG}: ${OBJS}
        ${CC} ${LDFLAGS} ${OBJS} -o ${PROG}

clean:
        rm -f ${PROG} *.o
