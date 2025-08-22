// DropZone.tsx – minimal repro / fix
import React, { useMemo, useRef, useState, useEffect } from 'react';

export function DropZone({
  accept = [],
  multiple = true,
  isReadonly = false,
  onFiles,
}: {
  accept?: string[];
  multiple?: boolean;
  isReadonly?: boolean;
  onFiles: (files: File[]) => void;
}) {
  const inputRef = useRef<HTMLInputElement>(null);
  const dragCounter = useRef(0);
  const [isDragging, setDragging] = useState(false);

  const acceptAttr = useMemo(
    () => (accept.length ? accept.join(',') : undefined),
    [accept]
  );

  const onlyFiles = (e: React.DragEvent) =>
    Array.from(e.dataTransfer?.types ?? []).includes('Files');

  const onDragEnter = (e: React.DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();
    if (!onlyFiles(e) || isReadonly) return;
    dragCounter.current += 1;
    setDragging(true);
    console.log('[dropzone] dragenter', dragCounter.current);
  };

  const onDragOver = (e: React.DragEvent<HTMLDivElement>) => {
    // REQUIRED or drop won't fire
    e.preventDefault();
    e.stopPropagation();
    if (!onlyFiles(e) || isReadonly) return;
    e.dataTransfer.dropEffect = 'copy';
    // no state change here; this fires a lot
  };

  const onDragLeave = (e: React.DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();
    // Many browsers fire dragleave when moving between children → use counter
    dragCounter.current = Math.max(0, dragCounter.current - 1);
    console.log('[dropzone] dragleave', dragCounter.current);
    if (dragCounter.current === 0) setDragging(false);
  };

  const onDrop = (e: React.DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();
    dragCounter.current = 0;
    setDragging(false);
    if (isReadonly) return;
    const list = e.dataTransfer.files;
    if (!list || list.length === 0) return;
    const arr = Array.from(list);
    if (!multiple && arr.length > 1) arr.splice(1);
    onFiles(arr);
  };

  // Optional safety: prevent the WINDOW from hijacking drops outside the box
  useEffect(() => {
    const prevent = (evt: DragEvent) => {
      // do not block if you specifically want to drop outside
      evt.preventDefault();
    };
    window.addEventListener('dragover', prevent);
    window.addEventListener('drop', prevent);
    return () => {
      window.removeEventListener('dragover', prevent);
      window.removeEventListener('drop', prevent);
    };
  }, []);

  return (
    <div
      style={{
        border: '2px dashed #86b7ff',
        background: isDragging ? '#eef6ff' : '#f9fbff',
        borderRadius: 8,
        minHeight: 140,
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        gap: 8,
        cursor: isReadonly ? 'not-allowed' : 'pointer',
        position: 'relative',
      }}
      // Capture phase helps beat the parent’s handlers
      onDragEnterCapture={onDragEnter}
      onDragOverCapture={onDragOver}
      onDragLeaveCapture={onDragLeave}
      onDropCapture={onDrop}
      onClick={() => !isReadonly && inputRef.current?.click()}
      role="button"
      aria-disabled={isReadonly}
    >
      <span aria-hidden>📎</span>
      {isReadonly ? (
        <span style={{ color: '#888' }}>Uploaded Files</span>
      ) : (
        <span>
          <strong>Drag and drop</strong> or <u>Browse Files</u>
        </span>
      )}
      <input
        ref={inputRef}
        type="file"
        accept={acceptAttr}
        multiple={multiple}
        onChange={(e) => {
          if (isReadonly) return;
          const list = e.target.files;
          if (!list || list.length === 0) return;
          const arr = Array.from(list);
          if (!multiple && arr.length > 1) arr.splice(1);
          onFiles(arr);
          // allow re-picking the same file
          e.currentTarget.value = '';
        }}
        // Invisible input should NOT intercept pointer/drag
        style={{ position: 'absolute', width: 0, height: 0, opacity: 0, pointerEvents: 'none' }}
      />
    </div>
  );
}




// src/components/UploadBox/UploadBox.tsx
import React, { useMemo, useRef, useState } from 'react';
import styles from './UploadBox.module.css';
import type { UploadedFile } from './types';

export type UploadBoxProps = {
  title?: string;
  /** Array of MIME types or extensions (e.g., ['application/pdf','.doc','.docx']) */
  accept?: string[];
  multiple?: boolean;
  maxSizeBytes?: number;

  /** Parent-controlled list of files (existing + in-flight) */
  files?: UploadedFile[];

  /** Readonly mode disables DnD, browse & delete */
  isReadonly?: boolean;

  /** Hide/show progress bar + percent (default true) */
  showProgressBar?: boolean;

  /** Events (all server work happens in parent) */
  onPickFiles?: (files: File[]) => void;         // when user selects/drops files
  onRemove?: (file: UploadedFile) => void;       // when user clicks remove
};

const fmtKB = (bytes: number) => `${Math.round(bytes / 1024)} KB`;

export const UploadBox: React.FC<UploadBoxProps> = ({
  title = 'RAMP Document',
  accept = [],
  multiple = true,
  maxSizeBytes,
  files = [],
  isReadonly = false,
  showProgressBar = true,
  onPickFiles,
  onRemove,
}) => {
  const inputRef = useRef<HTMLInputElement>(null);
  const [isDragging, setDragging] = useState(false);

  /** Build input accept attribute from array */
  const acceptAttr = useMemo(
    () => (accept && accept.length ? accept.join(',') : undefined),
    [accept]
  );

  const handleInputChange = (list: FileList | null) => {
    if (!list || isReadonly) return;
    const inArr = Array.from(list);

    // optional client-side size filter (just a guard; real validation should be server-side)
    const filtered = typeof maxSizeBytes === 'number'
      ? inArr.filter(f => f.size <= maxSizeBytes)
      : inArr;

    if (!multiple && filtered.length > 1) filtered.splice(1);
    onPickFiles?.(filtered);
    // reset input so same file can be picked again
    if (inputRef.current) inputRef.current.value = '';
  };

  const handleDrop = (e: React.DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    setDragging(false);
    if (isReadonly) return;
    handleInputChange(e.dataTransfer.files);
  };

  return (
    <div className={styles.wrapper}>
      {/* Title */}
      <div className={styles.label}>{title}</div>

      {/* Dropzone */}
      <div
        className={`${styles.dropzone} ${isDragging ? styles.dragging : ''} ${isReadonly ? styles.readonly : ''}`}
        onDragOver={(e) => {
          if (isReadonly) return;
          e.preventDefault();
          setDragging(true);
        }}
        onDragLeave={() => !isReadonly && setDragging(false)}
        onDrop={handleDrop}
        onClick={() => !isReadonly && inputRef.current?.click()}
        role="button"
        aria-disabled={isReadonly}
      >
        <span className={styles.paperclip} aria-hidden>📎</span>
        {isReadonly ? (
          <span className={styles.ctaMuted}>Uploaded Files</span>
        ) : (
          <span className={styles.cta}>
            <strong>Drag and drop</strong> or <span className={styles.browse}>Browse Files</span>
          </span>
        )}
        <input
          ref={inputRef}
          type="file"
          accept={acceptAttr}
          multiple={multiple}
          className={styles.input}
          onChange={(e) => handleInputChange(e.target.files)}
          disabled={isReadonly}
        />
      </div>

      {/* File list (fully parent-controlled) */}
      {files?.length > 0 && (
        <ul className={styles.list} aria-live="polite">
          {files.map((f) => {
            const pct = f.progress ?? (f.status === 'done' ? 100 : 0);
            return (
              <li key={(f.id ?? '') + ':' + f.name} className={styles.item}>
                <div className={styles.row}>
                  <div className={styles.meta}>
                    <span className={styles.badge}>DOC</span>
                    <div>
                      <div className={styles.name}>
                        {f.url ? (
                          <a href={f.url} target="_blank" rel="noreferrer">{f.name}</a>
                        ) : (
                          f.name
                        )}
                      </div>
                      <div className={styles.size}>{fmtKB(f.size)}</div>
                    </div>
                  </div>

                  <div className={styles.controls}>
                    {showProgressBar && (
                      <div className={styles.percent}>
                        {f.status === 'done' ? '100%' : `${pct}%`}
                      </div>
                    )}
                    {f.status === 'done' && (
                      <span className={styles.check} aria-label="Uploaded">✔️</span>
                    )}
                    {!isReadonly && (
                      <button
                        type="button"
                        className={styles.trash}
                        onClick={() => onRemove?.(f)}
                        aria-label={`Remove ${f.name}`}
                        title="Remove"
                      >
                        🗑️
                      </button>
                    )}
                  </div>
                </div>

                {showProgressBar && (
                  <div className={styles.progressBarOuter}>
                    <div
                      className={styles.progressBarInner}
                      style={{ width: `${pct}%` }}
                    />
                  </div>
                )}

                {f.status === 'error' && (
                  <div className={styles.error}>⚠️ {f.error || 'Upload failed'}</div>
                )}
              </li>
            );
          })}
        </ul>
      )}
    </div>
  );
};

// src/store/uploadApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export type UploadedFile = {
  id: string;
  name: string;
  size: number;        // bytes
  url?: string;
  ext?: string;
};

export const uploadApi = createApi({
  reducerPath: 'uploadApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Uploads'],
  endpoints: (builder) => ({
    // List previously uploaded files (for initial render)
    listDocuments: builder.query<UploadedFile[], void>({
      query: () => '/uploads',
      providesTags: (result) =>
        result ? [...result.map((f) => ({ type: 'Uploads' as const, id: f.id })), { type: 'Uploads', id: 'LIST' }] : [{ type: 'Uploads', id: 'LIST' }],
    }),

    // Delete by id
    deleteDocument: builder.mutation<{ success: boolean }, { id: string }>({
      query: ({ id }) => ({ url: `/upload/${id}`, method: 'DELETE' }),
      invalidatesTags: (_r, _e, arg) => [{ type: 'Uploads', id: arg.id }, { type: 'Uploads', id: 'LIST' }],
    }),

    // Upload with progress (XMLHttpRequest)
    uploadDocument: builder.mutation<
      UploadedFile,
      { file: File; onProgress?: (percent: number) => void }
    >({
      async queryFn({ file, onProgress }) {
        try {
          const form = new FormData();
          form.append('file', file);

          const xhr = new XMLHttpRequest();
          const promise = new Promise<UploadedFile>((resolve, reject) => {
            xhr.open('POST', '/api/upload');
            xhr.responseType = 'json';

            xhr.upload.onprogress = (evt) => {
              if (!evt.lengthComputable) return;
              const pct = Math.round((evt.loaded / evt.total) * 100);
              onProgress?.(pct);
            };

            xhr.onload = () => {
              if (xhr.status >= 200 && xhr.status < 300) {
                resolve(xhr.response as UploadedFile);
              } else {
                reject({
                  status: xhr.status,
                  message: xhr.response?.message || 'Upload failed',
                });
              }
            };
            xhr.onerror = () => reject({ status: xhr.status, message: 'Network error' });

            xhr.send(form);
          });

          const data = await promise;
          onProgress?.(100);
          return { data };
        } catch (error: any) {
          return { error };
        }
      },
      // tell RTKQ to refresh list after successful upload
      invalidatesTags: [{ type: 'Uploads', id: 'LIST' }],
    }),
  }),
});

export const {
  useListDocumentsQuery,
  useDeleteDocumentMutation,
  useUploadDocumentMutation,
} = uploadApi;
