---
title: IdeaYGnuPG
date: 2009-03-02 12:00 UTC
tags:
---
## PLUGIN PARA GNUPG QUE SOPORTA ALGORITMO IDEA

GnuPG desde la versión 1.0.5 (ver {1}) no soporta el algoritmo de encripción IDEA por ser patentado aunque es el estándar en PGP (en particular pgpi 5.0).

! INSTALACIÓN Y CONFIGURACIÓN

Para agregar soporte a GnuPG debe:

* Descargar idea.c e idea.h.  Puede ser de http://www.nosneros.net/hso/code/gnupg_idea_patches/ (al final de este artículo también están).
* Compilar con: 
<pre>
gcc -Wall -O2 -shared -fPIC -o idea idea.c
mkdir ~/lib/
cp idea ~/lib/
</pre>
*  Configurar GnuPG: en el archivo ~/.gnupg/gpg.conf agregar al final la línea:
<pre>
load-extension /home/miusuario/lib/idea
</pre>


##REFERENCIAS

* {1} http://www.spywarewarrior.com/uiuc/gpg-idea/gpg-idea.htm
* {2} http://www.nosneros.net/hso/code/gnupg_idea_patches/

----
Esta información se dedica a Jesucristo por su Paciencia y se libera al dominio público. vtamara@pasosdeJesus.org 2008.


-----
! idea.h
<pre>
const char *
idea_get_info( int algo, size_t *keylen,
                   size_t *blocksize, size_t *contextsize,
                   int  (**r_setkey)( void *c, byte *key, unsigned keylen ),
                   void (**r_encrypt)( void *c, byte *outbuf, byte *inbuf ),
                   void (**r_decrypt)( void *c, byte *outbuf, byte *inbuf )
                 );

</pre>
-----
! idea.c

<pre>
/* idea.c  -  IDEA algorithm
 * Copyright (c) 1997, 1998, 1999, 2001, 2002 by Werner Koch (dd9jn)
 *
 * Permission is hereby granted, free of charge, to any person obtaining a
 * copy of this software and associated documentation files (the "Software"),
 * to deal in the Software without restriction, including without limitation
 * the rights to use, copy, modify, merge, publish, distribute, sublicense,
 * and/or sell copies of the Software, and to permit persons to whom the
 * Software is furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
 * WERNER KOCH BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
 * IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
 * CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 *
 * Except as contained in this notice, the name of Werner Koch shall not be
 * used in advertising or otherwise to promote the sale, use or other dealings
 * in this Software without prior written authorization from Werner Koch.
 *
 * DUE TO PATENT CLAIMS THE DISTRIBUTION OF THE SOFTWARE IS NOT ALLOWED IN
 * THESE COUNTRIES: 
 *     AUSTRIA, FRANCE, GERMANY, ITALY, JAPAN, THE NETHERLANDS, 
 *     SPAIN, SWEDEN, SWITZERLAND, THE UK AND THE US.
 */

/*
 * Please see http://www.noepatents.org/ to learn why software patents
 * are bad for society and what you can do to fight them.
 *
 * The code herein is based on the one from:
 *   Bruce Schneier: Applied Cryptography. John Wiley & Sons, 1996.
 *    ISBN 0-471-11709-9. .
 *
 * How to compile:
       gcc -Wall -O2 -shared -fPIC -o idea idea.c
 *
 * 2001-06-08 wk  Changed distribution conditions
 * 2001-06-11 wk  Fixed invert_key (which is not used in CFB mode) 
 *                Thanks to Mark A. Borgerding.  Added definition for
 *                the PowerPC.
 * 2002-08-03 wk  Removed dependeny to g10_log_fatal and perform selftest
 *                in idea_get_info.
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

/* configuration stuff */
#ifdef __alpha__
  #define !SIZEOF_UNSIGNED_LONG 8
#else
  #define !SIZEOF_UNSIGNED_LONG 4
#endif

#if defined(__mc68000__) || defined (__sparc__) || defined (__PPC__) \
    || (defined(__mips__) && (defined(MIPSEB) || defined (__MIPSEB__)) ) \
    || defined(__powerpc__) \
    || defined(__hpux__) /* should be replaced by the Macro for the PA */
  #define !BIG_ENDIAN_HOST 1
#else
  #define !LITTLE_ENDIAN_HOST 1
#endif

typedef unsigned long  ulong;
typedef unsigned short ushort;
typedef unsigned char  byte;

typedef unsigned short u16;
typedef unsigned long  u32;

/* end configurable stuff */

#ifndef DIM
  #define DIM(v) (sizeof(v)/sizeof((v)![0]))
  #define DIMof(type,member)   DIM(((type *)0)->member)
#endif

/* imports */
void g10_log_fatal( const char *fmt, ... );


/* local stuff */

#define FNCCAST_SETKEY(f)  ((int(*)(void*, byte*, unsigned))(f))
#define FNCCAST_CRYPT(f)   ((void(*)(void*, byte*, byte*))(f))

#define IDEA_KEYSIZE 16
#define IDEA_BLOCKSIZE 8
#define IDEA_ROUNDS 8
#define IDEA_KEYLEN (6*IDEA_ROUNDS+4)

typedef struct {
    u16 ek![IDEA_KEYLEN];
    u16 dk![IDEA_KEYLEN];
    int have_dk;
} IDEA_context;


static int do_setkey( IDEA_context *c, byte *key, unsigned keylen );
static void encrypt_block( IDEA_context *bc, byte *outbuf, byte *inbuf );
static void decrypt_block( IDEA_context *bc, byte *outbuf, byte *inbuf );
static int selftest(int);



static u16
mul_inv( u16 x )
{
    u16 t0, t1;
    u16 q, y;

    if( x < 2 )
	return x;
    t1 = 0x10001L / x;
    y =  0x10001L % x;
    if( y ``` 1 )
	return (1-t1) & 0xffff;

    t0 = 1;
    do {
	q = x / y;
	x = x % y;
	t0 += q * t1;
	if( x ``` 1 )
	    return t0;
	q = y / x;
	y = y % x;
	t1 += q * t0;
    } while( y != 1 );
    return (1-t1) & 0xffff;
}



static void
expand_key( byte *userkey, u16 *ek )
{
    int i,j;

    for(j=0; j < 8; j++ ) {
	ek![j] = (*userkey << 8) + userkey![1];
	userkey += 2;
    }
    for(i=0; j < IDEA_KEYLEN; j++ ) {
	i++;
	ek![i+7] = ek![i&7] << 9 | ek![(i+1)&7] >> 7;
	ek += i & 8;
	i &= 7;
    }
}


static void
invert_key( u16 *ek, u16 dk![IDEA_KEYLEN] )
{
    int i;
    u16 t1, t2, t3;
    u16 temp![IDEA_KEYLEN];
    u16 *p = temp + IDEA_KEYLEN;

    t1 = mul_inv( *ek++ );
    t2 = -*ek++;
    t3 = -*ek++;
    *--p = mul_inv( *ek++ );
    *--p = t3;
    *--p = t2;
    *--p = t1;

    for(i=0; i < IDEA_ROUNDS-1; i++ ) {
	t1 = *ek++;
	*--p = *ek++;
	*--p = t1;

	t1 = mul_inv( *ek++ );
	t2 = -*ek++;
	t3 = -*ek++;
	*--p = mul_inv( *ek++ );
	*--p = t2;
	*--p = t3;
	*--p = t1;
    }
    t1 = *ek++;
    *--p = *ek++;
    *--p = t1;

    t1 = mul_inv( *ek++ );
    t2 = -*ek++;
    t3 = -*ek++;
    *--p = mul_inv( *ek++ );
    *--p = t3;
    *--p = t2;
    *--p = t1;
    memcpy(dk, temp, sizeof(temp) );
    memset(temp, 0, sizeof(temp) );  /* burn temp */
}


static void
cipher( byte *outbuf, byte *inbuf, u16 *key )
{
    u16 x1, x2, x3,x4, s2, s3;
    u16 *in, *out;
    int r = IDEA_ROUNDS;
  #define MUL(x,y) \
	do {u16 _t16; u32 _t32; 		    \
	    if( (_t16 = (y)) ) {		    \
		if( (x = (x)&0xffff) ) {	    \
		    _t32 = (u32)x * _t16;	    \
		    x = _t32 & 0xffff;		    \
		    _t16 = _t32 >> 16;		    \
		    x = ((x)-_t16) + (x<_t16?1:0);  \
		}				    \
		else {				    \
		    x = 1 - _t16;		    \
		}				    \
	    }					    \
	    else {				    \
		x = 1 - x;			    \
	    }					    \
	} while(0)

    in = (u16*)inbuf;
    x1 = *in++;
    x2 = *in++;
    x3 = *in++;
    x4 = *in;
  #ifdef !LITTLE_ENDIAN_HOST
    x1 = (x1>>8) | (x1<<8);
    x2 = (x2>>8) | (x2<<8);
    x3 = (x3>>8) | (x3<<8);
    x4 = (x4>>8) | (x4<<8);
  #endif
    do {
	MUL(x1, *key++);
	x2 += *key++;
	x3 += *key++;
	MUL(x4, *key++ );

	s3 = x3;
	x3 ^= x1;
	MUL(x3, *key++);
	s2 = x2;
	x2 ^=x4;
	x2 += x3;
	MUL(x2, *key++);
	x3 += x2;

	x1 ^= x2;
	x4 ^= x3;

	x2 ^= s3;
	x3 ^= s2;
    } while( --r );
    MUL(x1, *key++);
    x3 += *key++;
    x2 += *key++;
    MUL(x4, *key);

    out = (u16*)outbuf;
  #ifdef !LITTLE_ENDIAN_HOST
    *out++ = (x1>>8) | (x1<<8);
    *out++ = (x3>>8) | (x3<<8);
    *out++ = (x2>>8) | (x2<<8);
    *out   = (x4>>8) | (x4<<8);
  #else
    *out++ = x1;
    *out++ = x3;
    *out++ = x2;
    *out   = x4;
  #endif
  #undef MUL
}


static int
do_setkey( IDEA_context *c, byte *key, unsigned keylen )
{
    assert(keylen ``` 16);
    c->have_dk = 0;
    expand_key( key, c->ek );
    invert_key( c->ek, c->dk );
    return 0;
}

static void
encrypt_block( IDEA_context *c, byte *outbuf, byte *inbuf )
{
    cipher( outbuf, inbuf, c->ek );
}

static void
decrypt_block( IDEA_context *c, byte *outbuf, byte *inbuf )
{
    if( !c->have_dk ) {
       c->have_dk = 1;
       invert_key( c->ek, c->dk );
    }
    cipher( outbuf, inbuf, c->dk );
}


static int
selftest( int check_decrypt )
{
static struct {
    byte key![16];
    byte plain![8];
    byte cipher![8];
} test_vectors[] = {
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0x00, 0x00, 0x00, 0x01, 0x00, 0x02, 0x00, 0x03 },
      { 0x11, 0xFB, 0xED, 0x2B, 0x01, 0x98, 0x6D, 0xE5 } },
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08 },
      { 0x54, 0x0E, 0x5F, 0xEA, 0x18, 0xC2, 0xF8, 0xB1 } },
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0x00, 0x19, 0x32, 0x4B, 0x64, 0x7D, 0x96, 0xAF },
      { 0x9F, 0x0A, 0x0A, 0xB6, 0xE1, 0x0C, 0xED, 0x78 } },
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0xF5, 0x20, 0x2D, 0x5B, 0x9C, 0x67, 0x1B, 0x08 },
      { 0xCF, 0x18, 0xFD, 0x73, 0x55, 0xE2, 0xC5, 0xC5 } },
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0xFA, 0xE6, 0xD2, 0xBE, 0xAA, 0x96, 0x82, 0x6E },
      { 0x85, 0xDF, 0x52, 0x00, 0x56, 0x08, 0x19, 0x3D } },
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0x0A, 0x14, 0x1E, 0x28, 0x32, 0x3C, 0x46, 0x50 },
      { 0x2F, 0x7D, 0xE7, 0x50, 0x21, 0x2F, 0xB7, 0x34 } },
    { { 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, 0x00, 0x04,
	0x00, 0x05, 0x00, 0x06, 0x00, 0x07, 0x00, 0x08 },
      { 0x05, 0x0A, 0x0F, 0x14, 0x19, 0x1E, 0x23, 0x28 },
      { 0x7B, 0x73, 0x14, 0x92, 0x5D, 0xE5, 0x9C, 0x09 } },
    { { 0x00, 0x05, 0x00, 0x0A, 0x00, 0x0F, 0x00, 0x14,
	0x00, 0x19, 0x00, 0x1E, 0x00, 0x23, 0x00, 0x28 },
      { 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08 },
      { 0x3E, 0xC0, 0x47, 0x80, 0xBE, 0xFF, 0x6E, 0x20 } },
    { { 0x3A, 0x98, 0x4E, 0x20, 0x00, 0x19, 0x5D, 0xB3,
	0x2E, 0xE5, 0x01, 0xC8, 0xC4, 0x7C, 0xEA, 0x60 },
      { 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08 },
      { 0x97, 0xBC, 0xD8, 0x20, 0x07, 0x80, 0xDA, 0x86 } },
    { { 0x00, 0x64, 0x00, 0xC8, 0x01, 0x2C, 0x01, 0x90,
	0x01, 0xF4, 0x02, 0x58, 0x02, 0xBC, 0x03, 0x20 },
      { 0x05, 0x32, 0x0A, 0x64, 0x14, 0xC8, 0x19, 0xFA },
      { 0x65, 0xBE, 0x87, 0xE7, 0xA2, 0x53, 0x8A, 0xED } },
    { { 0x9D, 0x40, 0x75, 0xC1, 0x03, 0xBC, 0x32, 0x2A,
	0xFB, 0x03, 0xE7, 0xBE, 0x6A, 0xB3, 0x00, 0x06 },
      { 0x08, 0x08, 0x08, 0x08, 0x08, 0x08, 0x08, 0x08 },
      { 0xF5, 0xDB, 0x1A, 0xC4, 0x5E, 0x5E, 0xF9, 0xF9 } }
};
    IDEA_context c;
    byte buffer![8];
    int i;

    for(i=0; i < DIM(test_vectors); i++ ) {
	do_setkey( &c, test_vectors![i].key, 16 );
	if( !check_decrypt ) {
	    encrypt_block( &c, buffer, test_vectors![i].plain );
	    if( memcmp( buffer, test_vectors![i].cipher, 8 ) )
              {
		fprintf (stderr, "idea encryption (%d) failed\n", i);
                return -1;
              }
	}
	else {
	    decrypt_block( &c, buffer, test_vectors![i].cipher );
	    if( memcmp( buffer, test_vectors![i].plain, 8 ) )
              {
		fprintf (stderr, "idea decryption (%d) failed\n", i);
                return -1;
              }
	}
    }
    return 0;
}


/*
 * Return some information about the algorithm.  We need algo here to
 * distinguish different flavors of the algorithm.
 * Returns: A pointer to string describing the algorithm or NULL if
 *	    the ALGO is invalid.
 */
const char *
idea_get_info( int algo, size_t *keylen,
		   size_t *blocksize, size_t *contextsize,
		   int	(**r_setkey)( void *c, byte *key, unsigned keylen ),
		   void (**r_encrypt)( void *c, byte *outbuf, byte *inbuf ),
		   void (**r_decrypt)( void *c, byte *outbuf, byte *inbuf )
		 )
{
    static int initialized = 0;

    if( !initialized ) {
	initialized = 1;
	if ( selftest(0) || selftest(1) )
          return NULL;
    }
    *keylen = 128;
    *blocksize = 8;
    *contextsize = sizeof(IDEA_context);
    *r_setkey = FNCCAST_SETKEY(do_setkey);
    *r_encrypt= FNCCAST_CRYPT(encrypt_block);
    *r_decrypt= FNCCAST_CRYPT(decrypt_block);
    if( algo ``` 1 )
	return "IDEA";
    return NULL;
}



const char * const gnupgext_version = "IDEA ($Revision: 1.11 $)";

static struct {
    int class;
    int version;
    int  value;
    void (*func)(void);
} func_table[] = {
    { 20, 1, 0, (void(*)(void))idea_get_info },
    { 21, 1, 1 },
};



/*
 * Enumerate the names of the functions together with informations about
 * this function. Set sequence to an integer with a initial value of 0 and
 * do not change it.
 * If what is 0 all kind of functions are returned.
 * Return values: class := class of function:
 *			   10 = message digest algorithm info function
 *			   11 = integer with available md algorithms
 *			   20 = cipher algorithm info function
 *			   21 = integer with available cipher algorithms
 *			   30 = public key algorithm info function
 *			   31 = integer with available pubkey algorithms
 *		  version = interface version of the function/pointer
 *			    (currently this is 1 for all functions)
 */
void *
gnupgext_enum_func( int what, int *sequence, int *class, int *vers )
{
    void *ret;
    int i = *sequence;

    do {
	if( i >= DIM(func_table) || i < 0 ) {
	    return NULL;
	}
	*class = func_table![i].class;
	*vers  = func_table![i].version;
	switch( *class ) {
	  case 11:
	  case 21:
	  case 31:
	    ret = &func_table![i].value;
	    break;
	  default:
	    ret = (void*)func_table![i].func;
	    break;
	}
	i++;
    } while( what && what != *class );

    *sequence = i;
    return ret;
}

</pre>
