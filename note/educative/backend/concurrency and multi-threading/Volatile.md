# Volatile

# When volatile thread-safe?

Volatile comes into play because of multiples levels of memory in hardware architecture required for performance
enhancements. If there's a single thread that writes to the volatile variable and other threads only read the volatile
variable then just using volatile is enough, however, if there's a possibility of multiple threads writing to the
volatile variable then "synchronized" would be required to ensure atomic writes to the variable.