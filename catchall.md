# Catchall tips and tricks

- - -

# To see the assembly of any of the executables in Intel syntax:

objdump -M intel -d -r -j .text cst | less

- - -

# Postfix / mail setup:

TODO: Mail setup - apparently I used a script before to do this!

You can send an email from kuoney@gmail.com using the following command:

mail -s "subject" kuoney@gmail.com <<__EOF
> content
> __EOF
